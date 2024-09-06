## Description

The `RevertOptions` struct has 1 parameter that is not used properly: `bool callOnRevert`. This can damage users in a lot of unexpected ways due to calling a function on arbitrary contracts even if signaled not to do so.

A user signals `callOnRevert == false` in a cross-chain tx from Zetta chain to a spoke chain calling, for example, `depositAndCall()` on `GatewayZEVM` to transfer some `ZRC20`. 

Yet it reverts on spoke chain and thus `depositAndRevert()` in `GatewayZEVM` is called. This is so the user can get his funds back in origin chain.

Yet here `onRevert()` is always called regardless of `callOnRevert` value. So `onRevert()` will be called, and this is so arbitrary that the impact can be anything, from a simple revert to a user taking an undesired action in some DeFi protocol that makes him lose money.

## Proof Of Concept

```solidity
    function depositAndRevert(
        address zrc20,
        uint256 amount,
        address target,
        RevertContext calldata revertContext
    )
        external
        onlyFungible
        whenNotPaused
    { 
        if (zrc20 == address(0) || target == address(0)) revert ZeroAddress();
        if (amount == 0) revert InsufficientZRC20Amount();
        if (target == FUNGIBLE_MODULE_ADDRESS || target == address(this)) revert InvalidTarget();

        // 🟢👁️1️⃣ Code called by the `Observer` in case of revert to get user funds back
        if (!IZRC20(zrc20).deposit(target, amount)) revert ZRC20DepositFailed();
        
        // 🟢👁️2️⃣ You can see it always calls onRevert() hook
        UniversalContract(target).onRevert(revertContext);
    }
```

## Recommendation

Only call `onRevert()` if `callOnRevert` is `true`. Add the vlaue to the `RevertContext` struct and use it in all kinds of revert actions the codebase has.

```solidity
if(callOnRevert) {someAddress.onRevert(params);}
```
