### Summary

For a winner to claim its prize a cross-chain tx has to be done from Avalanche to Ethereum. This transaction requires LINK token in the contract to get paid. The admin can just withdraw LINK token with `withdrawTokens()` from the `WinnablesTicketManager` contract and avoid the winner from claiming its prize.

As it's stated in the contest readme, `The principles that must always remain true are: -Winnables admins cannot do anything to prevent a winner from withdrawing their prize`. See [here](https://github.com/sherlock-audit/2024-08-winnables-raffles?tab=readme-ov-file#q-please-discuss-any-design-choices-you-made).

### Root Cause

Admin can withdraw any amount of LINK token from the contract in Avalanche network without any restrictions.

### Internal pre-conditions

Does not apply.

### External pre-conditions

Does not apply.

### Attack Path

1. Winner is chosen an `propagateRaffleWinner()` is called.
2. Admin sees this and front-runs this transaction with `withdrawTokens()` to withdraw enough LINK token from the contract so as to not have enough to pay for CCIP fees.

### Impact

Broken, as per devs words: **a principle that must always remain true**.


### PoC

The protocol assumes any token in `WinnablesTicketManager` is an accidentally sent token that is why there is no special checks to prevent any token withdrawal, see [here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicketManager.sol#L292). Yet the LINK token it is not accidentally sent, it is a token that is used to pay for the cross-chain transaction fees and it can be used for admin abuse to the winner.

Notice that the winner can always send LINK himself to the contract so there is enough. But really that does not matter as long as admin front-runs it can withdraw enough LINK to make the tx fail.

### Mitigation

If the token being withdrawn is LINK, force the contract to have a minimum balance of LINK, if admin tries to withdraw more than that, revert the withdrawal.

```diff
    function withdrawTokens(address tokenAddress, uint256 amount) external onlyRole(0) {
        IERC20 token = IERC20(tokenAddress); 
        uint256 balance = token.balanceOf(address(this));
+       if(tokenAddress == LINK_ADDRESS && balance - amount < MINIMUM_LINK_BALANCE) revert InsufficientBalance();
        if (amount < balance) revert InsufficientBalance();
        token.safeTransfer(msg.sender, amount);
    }
```

It is true that the user can call the `transferAndCall()` on the LINK token and execute the transfer in 1 tx. As LINK is an **ERC-677**. Yet for this to be possible `WinnablesTicketManager` must implement an `onTokenTransfer()` that the LINK token would use and it does not implement it. This is another way of fixing this issue.