### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

## Summary

When doing a `crossChain()` swap without using sgEth as `bridgeToken` and using `WooRouterV2` for the `bridgeToken` to `toToken` swap, the external exchange fee is always charged even if using the `WooRouterV2`.

## Vulnerability Detail

As seen in the `_handleNativeReceived()` function on `WooCrossRouterV4` if the user does not use an external exchange the external exchange fee (`dstExternalFeeRate`) is not charged. However, if the user uses a cross-swap that involves the `_handleERC20Received()` function, the external exchange fee is always charged. Follow the numbers of the following code snippet to better understand the issue:

```solidity
    function _handleERC20Received(/*func params*/) internal {
        address msgSender = _msgSender();
        if (toToken == bridgedToken) {
          // code for when there is no need for swap
        } else {
            // Deduct the external swap fee
            // 🟢1️⃣ Fee is calculated for the usage of an external exchange
            uint256 fee = (bridgedAmount * dstExternalFeeRate) / FEE_BASE;
            // 🟢2️⃣ bridgedAmount is substracted the fee, this means the user will receive less bridgedAmount
            // cause is being charged the external fee
            bridgedAmount -= fee;

            TransferHelper.safeApprove(bridgedToken, address(wooRouter), bridgedAmount);
            // 🟢3️⃣ We enter in the else statement because we don't wanna use an external exchange
            if (dst1inch.swapRouter != address(0)) {
                // code for using external swap...
            } else {
                // 🟢4️⃣ We swap locally but bridgedAmount is used which has the external fee applied even
                // if we are not using an external exchange
                try wooRouter.swap(bridgedToken, toToken, bridgedAmount, minToAmount, payable(to), to) returns (
                    uint256 realToAmount
                ) {
                    // 🟢5️⃣ Call succeeds and we got charged the external fee even if we are not using an external exchange
                    emit WooCrossSwapOnDstChain(/*event params*/);
                } catch {
                    // 🟢6️⃣ Even if the swap fails we got charged the external fee as bridgedToken here still has the fee applied
                    TransferHelper.safeTransfer(bridgedToken, to, bridgedAmount);
                    emit WooCrossSwapOnDstChain(/*event params*/);
                }
            }
        }
    }

    // 🟢7️⃣ In the _handleNativeReceived() is handled propely inserting the `bridgedAmount -= fee;`
    // inside the if condition for the external exchange

        function _handleNativeReceived(/*func params*/) internal {
        address msgSender = _msgSender();
        if (toToken == ETH_PLACEHOLDER_ADDR) {
            // code for when there is no need for swap
        }
        // Swap required!
        IWETH(weth).deposit{value: bridgedAmount}();

        if (dst1inch.swapRouter != address(0)) {
            uint256 fee = (bridgedAmount * dstExternalFeeRate) / FEE_BASE;
            // 🟢8️⃣ Here is inside the external swap part
            uint256 swapAmount = bridgedAmount - fee;
        } else{
    // ret of code of the function...
```

## Impact

Users pay fees they should not be paying. Also this eliminates an incentive to use the WooFi swap option when doing cross-chain transactions with non sgETH bridge tokens as you will pay anyways the external fee.

## Code Snippet

- Correct line when uing handle native option: https://github.com/sherlock-audit/2024-03-woofi-swap/blob/main/WooPoolV2/contracts/CrossChain/WooCrossChainRouterV4.sol#L303

- Incorrect line when using any ERC20 otion: https://github.com/sherlock-audit/2024-03-woofi-swap/blob/main/WooPoolV2/contracts/CrossChain/WooCrossChainRouterV4.sol#L414

## Tool used

Manual Review

## Recommendation

At the `_handleERC20Received()` function move the `bridgedAmount -= fee;` operation inside the external exchange if condition: `if (dst1inch.swapRouter != address(0)) {}`. So it's only applied when using an external exchange.
