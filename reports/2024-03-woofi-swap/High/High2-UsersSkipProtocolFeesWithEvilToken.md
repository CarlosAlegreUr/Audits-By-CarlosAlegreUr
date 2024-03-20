### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

## Summary

Users can send valid tokens cross-chain without paying fees leveraing the creation of a fake token handcrafted, chosing it as the `toToken` for the `WooCrossChainRouterV4` and chosing to use **1inch** for the final swap in the destination chain.

## Vulnerability Detail

1️⃣ User decides to send cross-chain a valid `fromToken`.

2️⃣ User deploys a fake ERC20 `toToken` contrat and creates a valid DEX pool for that contract in the destination chain. This token will always revert if called by **1inch** router.

3️⃣ User makes a cross-chain swap with `fromToken` using `sgETH` or any other valid token as `bridgeToken` and using the fake token as `toToken`. Using **1inch** for the `bridgeToken` => `toToken` final swap.

4️⃣ The swap will revert but as the execution is in a `try-catch` statement execution will continue. User will skip protocol fees and later swap on its own the `bridgedToken` to the desired `toToken`. Completely undermining and avoiding the protocol fees.

> 🔔 **Notice** ℹ️: Creating the fake to-token and the pool so **1inch** can swap it might deem the attack unprofitable if you plan to use it only for 1 corss-chain swap. But long term, skipping fees is worth the cost of the set-up.

When eventually in the destination chain the `sgReceive()` triggers here we have 2 execution flows:

1️⃣ Bridge token was `sgETH`: Then `_handleNativeReceived()` will be called. As the `toToken` is not `ETH_PLACEHOLDER_ADDR` a swap will be done, and as said, **1inch** is chosen.

Now here is the trick, the `toToken` will revert on purpose when called by the **1inch** contract, but the tx will continue as it is in a `try-catch` statement, executing the catch part which is:

```solidity
catch {
    TransferHelper.safeApprove(weth, address(wooRouter), 0);
    TransferHelper.safeTransfer(weth, to, bridgedAmount);
    emit WooCrossSwapOnDstChain(/*event params*/);
}
```

You can see the user will receive `bridgedAmount` but this amount has not been deducted the fees. Thus the user will have received `WETH` cross-chain skipping a part of the protocol's fees. Now if the user didn't want `WETH` in the destination chain that doesn't really matter they themselves can go to **1inch** and swap it only paying the **1inch** fees.


2️⃣ If bridge token was any other valid ERC20: Something similar would happen but in the `_handleERC20Received()` function. Then `toToken != bridgedToken` thus we enter in the swap part of the code, as chosen **1inch** will be used and the same revert trick will be applied. This code is also in a `try-catch` statement and the catch part is:

```solidity
catch {
bridgedAmount += fee;
TransferHelper.safeTransfer(bridgedToken, to, bridgedAmount);
emit WooCrossSwapOnDstChain(/*event params*/);
}
```

In this execution flow is even more clear that the `bridgedAmount` doesn't include the fees as the `fee` is added to it.

> 🔔 **Notice** ℹ️: Notice that the eventual swap after skipping the protocol's fee doesn't imply a second transaction as all could be coded inside the transfer function in the fake `toToken` which can detecet when is called by the `catch` on the `WooCrossChainRouterV4` and automatically execute the swap with no external fees on **1inch**.

> 🔔 **Notice** ℹ️: The fee is added because if the swap is not carried out then the protocol doesn't want you to charge a fee because it assumes you are being honest and something in a third-party out of control happened. But that is not what happened, here the user is the "evil-fees-skipper" that eventually actually makes an external swap because he doesn't want the `bridgedToken` so this completely undermines the protocol feature of charging a fee if expecting an external swap on the destination chain.

### Fees Skipper toToken Contract

<details>
<summary>View template code 🔍</summary>

The fee skipper contract would look something like this:

```solidity
contract FeeSkipper is ERC20 {
    function transfer(){
        if(calledBy1Inch){
            revert;
        }

        if(calledByWooCrossChainRouterV4){
            // swap on 1inch
            // transfer to user
        }
    }
}
```

</details>

## Impact

Protocol losses expected income source.

## Code Snippet

> 🚧 **Note** ⚠️: I didn't provide any executable code snippet as I couldnt find on the codebases any quickly reusable code to use the cross-chain router locally or on a testnet and I didn't have time to create one on my own. Instead I provide this clear and detailed **Vulnerability Details** as Proof Of Concept.

See try-catch statements code for external swaps in [this link](https://github.com/sherlock-audit/2024-03-woofi-swap/blob/main/WooPoolV2/contracts/CrossChain/WooCrossChainRouterV4.sol#L305).

And in [this one](https://github.com/sherlock-audit/2024-03-woofi-swap/blob/main/WooPoolV2/contracts/CrossChain/WooCrossChainRouterV4.sol#L418).

## Tool used

Manual Review

## Recommendation

Assume the users are being honest with the `toToken` they really want and always charge a fee on destination chain if a final swap has to be made.

But, track all the fees payed and caused in reverts and once a year analyze them and give them back to users that were honest and their revert was caused by external fators.

The way it's implemented now makes the external fee feature futile as anyone if they want can skip it.

This way if a user is lying about its intentions it still doesn't skip the procotol fees and the protocol doens't lose a revenue source.
