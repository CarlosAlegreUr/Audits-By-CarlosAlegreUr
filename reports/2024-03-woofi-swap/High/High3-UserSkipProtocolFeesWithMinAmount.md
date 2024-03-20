## Summary

Users can send valid tokens cross-chain without paying fees leveraing `minToAmount` parameter and `try-catch` blocks on the `WooCrossChainRouterV4`.

## Vulnerability Detail

1️⃣ User decides to send cross-chain with a `minToAmount` really small but != 0 so all the checks pass. Like `minToAmount` = 1. 

2️⃣ When the `sgReceive()` is eventually called on the destination chain. If chosen to do a swap from `bridgeToken` to `toToken`, if via **1inch**, as `minToAmount` is so small the `toAmount` swap offered by any exchange will be bigger and thus revert.

3️⃣ But because this exhanges calls are in a `try-catch` statement the tx wont stop and the `catch` part will be executed.

4️⃣ User will skip protocol external exchange fees and later swap on its own the `bridgedToken` to the desired `toToken`. Completely undermining and avoiding the protocol external exchange fees (`dstExternalFeeRate`).

When eventually in the destination chain the `sgReceive()` triggers here we have **2** execution flows. As we will see both of them en up with a **1inch** revert due to `minAmount` being too small:

1️⃣ If bridge token was `sgETH`: Then `_handleNativeReceived()` will be called. As the `toToken` is not `ETH_PLACEHOLDER_ADDR` a swap will be done, and as said, **1inch** is chosen.

Now here is the trick, the `toToken` will revert on purpose when called by the **1inch** contract, but the tx will continue as it is in a `try-catch` statement, executing the catch part which is:

```solidity
catch {
    TransferHelper.safeApprove(weth, address(wooRouter), 0);
    TransferHelper.safeTransfer(weth, to, bridgedAmount);
    emit WooCrossSwapOnDstChain(/*event params*/);
}
```

You can see the user will receive `bridgedAmount` but this amount has not been deducted the fees. Thus the user will have received `WETH` cross-chain skipping the protocol's fees. Now if the user didn't want `WETH` in the destination chain that doesn't really matter they themselves can go to **1inch** and swap it only paying the **1inch** fees.

2️⃣ If bridge token was any other valid ERC20: Something similar would happen but in the `_handleERC20Received()` function. Then in swaps which `toToken != bridgedToken` we enter in the swap part of the code, as chosen **1inch** will be used and the same revert trick will be applied. This code is also in a `try-catch` statement and the catch part is:

```solidity
catch {
bridgedAmount += fee;
TransferHelper.safeTransfer(bridgedToken, to, bridgedAmount);
emit WooCrossSwapOnDstChain(/*event params*/);
}
```

In this execution flow is even more clear that the `bridgedAmount` doesn't include the fees as the `fee` is added to it.

> 🔔 **Notice** ℹ️: The fee is added because if the swap is not carried out then the protocol doesn't want you to charge a fee because it assumes you are being honest and something in a third-party out of control happened. But that is not what happened, here the user is the "evil-fees-skipper" that eventually actually makes an external swap because he doesn't want the `bridgedToken` so this completely undermines the protocol feature of charging a fee if expecting an external swap on the destination chain.

## Impact

Protocol losses expected income source.

## Code Snippet

> 🚧 **Note** ⚠️: I didn't provide any executable code snippet as I couldnt find on the codebases any quickly reusable code to use the cross-chain router locally or on a testnet and I didn't have time to create one on my own. Instead I provide this clear and detailed **Vulnerability Details** as Proof Of Concept.

See try-catch statements code for external swaps in [this link](https://github.com/sherlock-audit/2024-03-woofi-swap/blob/main/WooPoolV2/contracts/CrossChain/WooCrossChainRouterV4.sol#L305).

And in [this one](https://github.com/sherlock-audit/2024-03-woofi-swap/blob/main/WooPoolV2/contracts/CrossChain/WooCrossChainRouterV4.sol#L418).

## Tool used

Manual Review

## Recommendation

Tell the users that if making a cross-chain tx that requires a swap in **1inch** on the destination chain they will always pay the external swap fee even if the tx reverts.

But, track all the fees payed and caused in reverts and once a year analyze them and give them back to users that were honest and their revert was caused by external fators.

The way it's implemented now makes the external fee feature futile as anyone if they want can skip it.