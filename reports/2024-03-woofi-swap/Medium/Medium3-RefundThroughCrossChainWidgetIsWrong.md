### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

## Summary

The refund mechanism of Stargate is not implemented correcly in the `WooCrossRouterForWidget` contract. If needed, the refund would go the WooFi team and not to the final user as it's intended. The WooFi team can always just send later the refund to the corresponding address but this is extra unnecesary and unexpected use of resources.

## Vulnerability Detail

Stargate has a refund mechanism just in case someone sends more ether than needed to pay for the cross-chain bridging fees. The problem is that, if `crossSwap()` at `WooCrossChainRouterV4` is called from `WooCrossRouterForWidget`, the refund is sent to the `WooCrossRouterForWidget` contract itself which its funds are controlled by the `owner` a.k.a. the WooFi protocol team.

As it's visible in the rest of the codebase the excess of money sent for fees is meant to go back to `msg.sender` which is expected to be the user making the cross-chain swap:

```solidity
// 1️⃣ At WooCrossRouterForWidget
// 2️⃣ crossSwap() calls => crossSwap() but on WooCrossChainRouterV4 which calls => 
//_bridgeByStargate() that calls => stargateRouter.swap{value: msgValue}(/*params*/)

    function crossSwap(/*func params*/) external payable whenNotPaused nonReentrant {
        // more code...
        // 3️⃣ As you can see, msg.sender in the exeution of `WooCrossChainRouterV4` will be `WooCrossRouterForWidget`
        crossRouter.crossSwap{value: msgValue}(refId, to, srcInfos, dstInfos, src1inch, dst1inch);
    }

    // 4️⃣ NOW AT WooCrossChainRouterV4
    function _bridgeByStargate(/*func params*/) internal {
        // more code...
        stargateRouter.swap{value: msgValue}(
            dstInfos.chainId, 
            sgInfo.sgPoolIds(sgInfo.sgChainIdLocal(), srcInfos.bridgeToken),
            sgInfo.sgPoolIds(dstInfos.chainId, dstInfos.bridgeToken), 
            payable(_msgSender()), // <= 5️⃣ rebate (refund) address in case of sending more money than needed for fees
            bridgeAmount,
            dstMinBridgeAmount,
            obj, 
            dstWooCrossChainRouter, 
            payload
        );
    }

    // 6️⃣ Notice! _msgSender() has not been overriden so it will work as the default version 
    // of OZ which is returning the default msg.sender from solidity. Which as seen in 3️⃣,
    // it will be WooCrossRouterForWidget

```

This is a problem because the contract will have to send the native coin back to the user or third party, which results in unnecesary unexpected expenses for the WooFi protocol.

## Impact

Undesired behaviour of the system and unexpected unnecesary extra expenses for the WooFi protocol if they wish to return the funds to their legitimate owner.

## Code Snippet

- [LINK: At WooCrossRouterForWidget crossSwap() calls crossSwap() at WooCrossChainRouterV4.](https://github.com/sherlock-audit/2024-03-woofi-swap/blob/main/WooPoolV2/contracts/CrossChain/WooCrossRouterForWidget.sol#L133)
- [LINK: At WooCrossChainRouterV4 crossSwap() calls _bridgeByStargate().](https://github.com/sherlock-audit/2024-03-woofi-swap/blob/main/WooPoolV2/contracts/CrossChain/WooCrossChainRouterV4.sol#L141)
- [LINK: At WooCrossChainRouterV4 _bridgeByStargate() calls to stargate router link.](https://github.com/sherlock-audit/2024-03-woofi-swap/blob/main/WooPoolV2/contracts/CrossChain/WooCrossChainRouterV4.sol#L260)

## Tool used

Manual Review

## Recommendation

Make the refund address a parameter of `_bridgeByStargate()` at `WooCrossChainRouterV4` instead of relying on any kind of `msg.sender`.
