## Relevant Context

When executing a `SopkeChain -> Zeta` chain transaction there are no fees charged for gas consumption in Zeta as the `FUNGIBLE_MODULE` is a special module at protocol level that does not pay gas fees. See this on a chat with the devs [here](https://cantina.xyz/code/80a33cf0-ad69-4163-a269-d27756aacb5e/comments#comment-2ae513c0-6910-4e11-9442-d8689469eac9), question Nº2.

## Description

If at destination the transaction reverts, the `Observer` will execute the revert tx option on origin, Spoke in this case. Yet this will consume gas that was not taken into account when paying the fees at the beggining of the cctx process.

Actually no fees were payed yet some should just in case of a revert.

## Impact Explanation

`TSS` loses money in the shape of gas every time a `Spoke -> Zeta` transaction reverts (`GatewayEVM`).

This is a big problem as a user can also make the tx revert on purpose and use a lot of gas in destination chain setting on `RevertOptions` a very high `onRevertGasLimit`.

## Proof of Concept

Lets use the `deposit()` and `executeRevert()` function pair as an example:

```solidity

    function deposit(
        address receiver,
        RevertOptions calldata revertOptions
    )
        external
        payable
        whenNotPaused
        nonReentrant
    {
        // ⏬👁️🟢1️⃣ See, no fees are payed in Spoke chain.
        if (msg.value == 0) revert InsufficientETHAmount();
        if (receiver == address(0)) revert ZeroAddress();
        
        (bool deposited,) = tssAddress.call{ value: msg.value }("");

        if (!deposited) revert DepositFailed();
        
        emit Deposited(msg.sender, receiver, msg.value, address(0), "", revertOptions);
    }

    // ⏬👁️🟢2️⃣ This func gets executed if revert on Zeta with `onRevertGasLimit` gas
    function executeRevert(
        address destination,
        bytes calldata data,
        RevertContext calldata revertContext
    )
        public
        payable
        onlyRole(TSS_ROLE)
        whenNotPaused
        nonReentrant
    {
        if (destination == address(0)) revert ZeroAddress();
        (bool success,) = destination.call{ value: msg.value }("");
        if (!success) revert ExecutionFailed();
        // ⏬👁️🟢3️⃣ Here you can execute whatever you want gas-free, sponsored by TSS
        Revertable(destination).onRevert(revertContext);
        
        emit Reverted(destination, address(0), msg.value, data, revertContext);
    }
```

## Recommendation

Take into account the `onRevertGasLimit` to charge a fee in case of reverts. And if it eventually does not revert, refund the user the extra amount that comes from that fee.
