## Relevant Context

It can happen that `ERC20Custody` is paused due to any issue on the code or a token that handles. Yet in this case it makes sense for the system not to pause the `GatewayEVM` as `ZetaConnector` txs would not be affected by issues on the `ERC20Custody` contract.

## Description

In the context provided, if someone calls `deposit()` or `depositAndCall()` with an ERC20 on `GatewayEVM` the tx would go through and the funds would be sent to the unsafe `ERC20Custody` contract.

You can clearly see that this is not safe and not the intention as other system contracts that also handle funds have a `whenNotPaused` modifier in their respective receive functions. Like `ZetaConnector` ones => `function receiveTokens(uint256 amount) external override whenNotPaused`.

The system due to the reason above clearly avoids sending funds in (and out) of the funds manager contract when they are paused.

Yet users won't be protected as if they call `deposit()` or `depositAndCall()` on the `GatewayEVM` the funds will be sent to the `ERC20Custody` contract because the only check made is if the token is whitelisted and not if the contrat you will send funds to is paused. Plus the `whitelisted()` function does not have a `whenNotPaused` modifier.

## Proof Of Concept

This is the `transferFromToAssetHandler()` code that gets executed from the `deposit()` and `depositAndCall()`:

```solidity
function transferFromToAssetHandler(address from, address token, uint256 amount) private {
    if (token == zetaToken) {
            // transfer to connector
            // transfer amount to gateway
            IERC20(token).safeTransferFrom(from, address(this), amount);
            // approve connector to handle tokens depending on connector version (eg. lock or burn)
            if (!IERC20(token).approve(zetaConnector, amount)) revert ApprovalFailed();
            // send tokens to connector
            // ⏬👁️🟢 has `whenNotPaused`. So if there is an issue with connector it reverts and all is safe. 
            ZetaConnectorBase(zetaConnector).receiveTokens(amount); modifier
    } else {
            // transfer to custody
            // ⏬👁️🟢 `whitelisted()` does not have `whenNotPaused` 
            if (!IERC20Custody(custody).whitelisted(token)) revert NotWhitelistedInCustody();
            IERC20(token).safeTransferFrom(from, custody, amount);
    }
}
```

## Impact Explanation

If the `ERC20Custody` contract is paused, the `GatewayEVM` can still send funds to it. This is not safe and the code clearly tries to protect users against these situations yet with ERC20s it does not.

## Recommendation

Add a check to see if the contract is paused, if so revert.

```diff
function transferFromToAssetHandler(address from, address token, uint256 amount) private {
    if (token == zetaToken) {
           //code...
    } else {
+           if(IERC20Custody(custody).paused()) revert CustodyPaused();
            // transfer to custody
            if (!IERC20Custody(custody).whitelisted(token)) revert NotWhitelistedInCustody();
            IERC20(token).safeTransferFrom(from, custody, amount);
    }
}
```
