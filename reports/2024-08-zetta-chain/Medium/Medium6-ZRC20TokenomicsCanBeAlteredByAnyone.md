## Description

In Zeta anyone can burn ZRC20 tokens. `burn()` has no access control modifier and can be called by anyone.

## Impact Explanation

If they burn them they will never be able to recover them, and the total supply that can be recovered in their origin chain will be reduced.

If people burn tokens the team can just mint them back using the `deposit()` function. But this is clearly not the expected use of `deposit()`. Tokenomics are broken, people are not supposed to be able to burn their tokens and the team is not supposed at all to be always supervising to mint people tokens back just in case they burn them. The purpose of that function is to mint tokens to people that initiated cross-chain transactions from any spoke chain.

Allowing people to burn their tokens has unexpected consequences (yet temporarily fixable until someone else decides to burn and we got the problem again) that break the expected tokenomics of the ZRC20 tokens' total supply availability and require costly constant monitoring and manual intervention to "fix" every time.

If a token is not burnable, and the protocol approves its ZRC20, it will be able to be burned and be effectively "paralized" as the user won't be able to withdraw that ZRC20 from the Zeta chain to the spoke chain as they are actually burned and any withdraw will revert due to user's insufficient balance.

## Proof of Concept

To burn their tokens people can just call `burn()` at `ZRC20.sol`:

```solidity
// ⏬👁️🟢 No access control modifier and external, anyone can call
function burn(uint256 amount) external override returns (bool) {
    _burn(msg.sender, amount);
    return true;
}

// ⏬👁️🟢 same in `_burn()`, no access control
function _burn(address account, uint256 amount) internal virtual {
    if (account == address(0)) revert ZeroAddress();

    uint256 accountBalance = _balances[account];
    if (accountBalance < amount) revert LowBalance();

    _balances[account] = accountBalance - amount;
    _totalSupply -= amount;

    emit Transfer(account, address(0), amount);
}
```

Usually the `burn()` will only be called from `_withdrawZRC20WithGasLimit()` in `GatewayZEVM` and after this events will be emitted to unlock the same amount in the corresponding spoke chain maintaining the same total supply.

Yet anyone can call this function, burn their tokens and mint no new tokens anywhere manipulating the `totalSupply`.

## Recommendation

Add an `onlyGateway` modifier. Give the `GatewayZEVM` some special `ACCESS_CONTROL` role for that.
