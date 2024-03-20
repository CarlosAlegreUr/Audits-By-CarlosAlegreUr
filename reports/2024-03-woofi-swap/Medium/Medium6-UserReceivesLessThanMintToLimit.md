### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

## Summary

In 2 cross-chain swap scenarios users using the protocol in an expected and valid way can receive less tokens than the desired marked minimum limit thus unwillingly lose funds. Even if the users lose funds, I rate this a Medium and not a High because the protocol has some capability (though not capable in all-scenarios) of fighting back this problem in case third-party services decide to increase their fees.

## Vulnerability Detail

When executing a cross-swap through the `crossSwap()` function at the `WooCrossChainRouterV4` with **sgETH** as `bridgeToken` and the native coin like ETH as the `toToken` (in the code this translates to `toToken == ETH_PLACEHOLDER_ADDR`) the user can receive less than the specified `minToAmount`. It can also happen with any `fromToken` and a `bridgeToken == toToken` but `bridgeToken != sgETH`.

## Impact

Users using the protocol in an expected and valid way will receive less tokens on cross-chain swaps than the desired marked minimum limit thus unwillingly lose funds.

## Code Snippet

> 🚧 **Note** ⚠️: I didn't provide any executable code snippet as I couldnt find in the codebase a quickly reusable code to use the cross-chain router locally or on a testnet and I didn't have time to create one on my own. Instead I provide this clear and very detaild execution code as _Proof Of Concept_.

> 🔔 **Notice** ℹ️: The only requirement for this to happen is that `(bridgeToken == sgEth && toToken == ETH_PLACEHOLDER_ADDR) || (fromToken == ANY_TOKEN && bridgeToken == toToken)`. Other simulated inputs in the explanation are specified to have a simpler execution flow to explain.

#### User calls `crossSwap()` at the `WooCrossChainRouterV4` contract:

- ***Inputs***:
```solidity
uint256 refId = validRefId, // Any (only used off-chain so it doesn't interfere in on-chain logic flow)
address payable to, // Destination address in destination chain owned by User
SrcInfos memory srcInfos, // fromToken == bridgeAsset && bridgeToken == sgInfo.sgETHs(sgInfo.sgChainIdLocal())
DstInfos calldata dstInfos, // Same bridgeToken as in srcInfos and toToken == ETH_PLACEHOLDER_ADDR
Src1inch calldata src1inch, // Any valid pool in source chain of fromToken to bridgeToken
Dst1inch calldata dst1inch // Doesn't matter won't execute
```

All the checks on the source chain will pass and the stargate protocol will initiate the cross-chain bridging of the bridge asset.
To see this is true follow the (🟢1️⃣) numbers that explain the execution in the crossSwap() function below:

<details> <summary> See detailed execution flow explanation 👁️ </summary>

```solidity
   function crossSwap(
        uint256 refId,
        address payable to,
        SrcInfos memory srcInfos,
        DstInfos calldata dstInfos,
        Src1inch calldata src1inch,
        Dst1inch calldata dst1inch
    ) external payable whenNotPaused nonReentrant {
        require(srcInfos.fromToken != address(0), "WooCrossChainRouterV3: !srcInfos.fromToken");
        require(
            dstInfos.toToken != address(0) && dstInfos.toToken != sgInfo.sgETHs(dstInfos.chainId),
            "WooCrossChainRouterV3: !dstInfos.toToken"
        );
        require(to != address(0), "WooCrossChainRouterV3: !to");

        uint256 msgValue = msg.value;
        uint256 bridgeAmount;
        uint256 fee = 0;

        {
            // Step 1: transfer
            // 🟢1️⃣ In our example fromToken == sgETH and not ETH_PLACEHOLDER_ADDR so we execute the else block
            if (srcInfos.fromToken == ETH_PLACEHOLDER_ADDR) {
                // code for when is native coin...
            } else {
                TransferHelper.safeTransferFrom(srcInfos.fromToken, msg.sender, address(this), srcInfos.fromAmount);
            }

            // 🟢2️⃣ In our example fromToken == srcInfos.bridgeToken so we execute the else block
            // Step 2: local swap by 1inch router
            if (srcInfos.fromToken != srcInfos.bridgeToken) {
                // 🟢3️⃣ Here goes the code for when srcInfos.fromToken != srcInfos.bridgeToken
                // this means a swap will be made whether through 1inch swap or WooRouterV2.
                // In the 1inch case the `bridgeAmount` will be returned with an already deducted fee
                // from 1inch, which we can't control.
                  if (src1inch.swapRouter != address(0)) {
                    TransferHelper.safeApprove(srcInfos.fromToken, address(wooRouter), srcInfos.fromAmount);
                    bridgeAmount = wooRouter.externalSwap(/*func args swaping fromAmount*/);
                    // 🟢4️⃣ Notice here is another fee, but this one is set by the protocol team so in case
                    // of needing to be lower so users are not damaged they could set it lower.
                    // Thus if taken this way `bridgeAmount` would be the amount received by the sawp from
                    // the fromAmount - 1inchFees - slippageOfTheSwap. Two factors the protocol cant control.
                    fee = (bridgeAmount * srcExternalFeeRate) / FEE_BASE;
                } else{
                    // WooRouterV2 swap code would go here...
                }
            } else {
                // 🟢5️⃣ Coming back to the execution flow of the example. This check passes as inputs set corretly 
                // but notice the `minToAmount` is not checked anywhere against `bridgeAmount`
                require(
                    srcInfos.fromAmount == srcInfos.minBridgeAmount, "WooCrossChainRouterV3: !srcInfos.minBridgeAmount"
                );
                bridgeAmount = srcInfos.fromAmount;
            }

            // 🟢6️⃣ Still `minToAmount` not checked
            require(
                bridgeAmount <= IERC20(srcInfos.bridgeToken).balanceOf(address(this)),
                "WooCrossChainRouterV3: !bridgeAmount"
            );
        }

        // Step 3: deduct the swap fee
        bridgeAmount -= fee;

        // Step 4: cross chain swap by StargateRouter
        // 🟢7️⃣ Notice!`bridgeAmount` is what we eventually bridge and there is no guarantees
        // after fees deduction that bridgeAmount >= minToAmount set by user. Sometimes exeution
        // flows, as seen, carry more fees than other but none makes sure bridgeAmount >= minToAmount.
        // So the user now would be sending a `bridgeAmount` > `minToAmount` expecting that, if
        // this would be the case something would stop and return him his funds. But it won't
        // happen as we will see now.
        // In _bridgeByStargate() there are also no checks of this kind:
        // you can see the code of this func here: 
        // https://github.com/sherlock-audit/2024-03-woofi-swap/blob/main/WooPoolV2/contracts/CrossChain/WooCrossChainRouterV4.sol#L219
        _bridgeByStargate(refId, to, msgValue, bridgeAmount, srcInfos, dstInfos, dst1inch);

        emit WooCrossSwapOnSrcChain(/*event params*/);
    }

    // 🟢8️⃣ When in destination chain sgReceive() will be activated and the Stargate router will
    // send the sgETH with the call to this function
    function sgReceive(
        uint16, 
        bytes memory, 
        uint256, 
        address bridgedToken,
        uint256 amountLD,
        bytes memory payload
    ) external {
        require(msg.sender == sgInfo.sgRouter(), "WooCrossChainRouterV3: INVALID_CALLER");

        (uint256 refId, address to, address toToken, uint256 minToAmount, Dst1inch memory dst1inch) =
            abi.decode(payload, (uint256, address, address, uint256, Dst1inch));

        // toToken won't be SGETH, and bridgedToken won't be ETH_PLACEHOLDER_ADDR
        // 🟢9️⃣ See we used sgEths so _handleNativeReceived() will be executed
        if (bridgedToken == sgInfo.sgETHs(sgInfo.sgChainIdLocal())) {
            // bridgedToken is SGETH, received native token
            _handleNativeReceived(refId, to, toToken, amountLD, minToAmount, dst1inch);
        } else {
            // 🟢1️⃣0️⃣ This won't be executed, once _handleNativeReceived() returns all is over
        }
    }

    // 🟢1️⃣1️⃣ And finally we are in _handleNativeReceived()
    function _handleNativeReceived(
        uint256 refId,
        address to,
        address toToken,
        uint256 bridgedAmount,
        uint256 minToAmount,
        Dst1inch memory dst1inch
    ) internal {
        address msgSender = _msgSender();
        if (toToken == ETH_PLACEHOLDER_ADDR) {
            // Directly transfer ETH
            // 🟢1️⃣2️⃣ bridgedAmount which was > minToAmount is transfered and the return will execute and all
            // will finish. The user will receive less toToken than he was willing to pay for.
            TransferHelper.safeTransferETH(to, bridgedAmount);
            emit WooCrossSwapOnDstChain(/*event params*/);
            return;
        }
        // 🟢1️⃣3️⃣ Rest of code wont be exeuted as toToken == ETH_PLACEHOLDER_ADDR.
        // more code...
    }

    // 🟢1️⃣4️⃣ And finally notice that if we would have made bridgeToken == toToken and taken the
    // _handleERC20Received() path, the same would have happened. No checks and directly transfered
    // the bridgedAmount to the user.
        function _handleERC20Received(/*func args*/) internal {
        address msgSender = _msgSender();
        if (toToken == bridgedToken) {
            TransferHelper.safeTransfer(bridgedToken, to, bridgedAmount);
            emit WooCrossSwapOnDstChain(/*event args*/);
        } 
```

See all code analyzed on the [WooCrossChainRouterV4.sol](https://github.com/sherlock-audit/2024-03-woofi-swap/blob/main/WooPoolV2/contracts/CrossChain/WooCrossChainRouterV4.sol#L25).


</details>

## Tool used

Manual Review

## Recommendation

At `crossSwap()` before sending the bridged token to the user. If the `bridgedToken` can be easily compared with the `toToken` as with the sgETH example exposed, then compare `minToAmount >= bridgedAmount` 

If not calculate the conversion of `bridgeToken` and `toToken` to the same accounting unit (USD for example) and compare them.

Dont allow cases were fees add up and  `minToAmount < bridgedAmount` to keep executing as once cross-chain swap is sent users have no capacity to cancel it. 
