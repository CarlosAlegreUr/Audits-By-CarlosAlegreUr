## Vulnerability Details

The `PriorityPool::withdraw()` makes the user specify how much **stLINK** to withdraw, which is `1:1` with **LINK**. Yet in the `WithdrawalPool:queueWithdrawal()`, this amount is tracked in shares.

So eventually when the user calls `WithdrawalPool::withdraw()`, the calculation of how much **LINK** to withdraw is done so with the **stakePerShare** price at time of when a `_finalizeWithdrawals()` call finalized the user's withdrawal request. See [here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/core/priorityPool/WithdrawalPool.sol#L283) and [here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/core/priorityPool/WithdrawalPool.sol#L448).

Notice that since user first request withdrawal until a `_finalizeWithdrawals()` is called, time can pass and thus those shares will appreciate in value as rewards are accrued. And eventually in the `WithdrawalPool::withdraw()` the value of the shares at the time of `_finalizeWithdrawals` will be used instead of the value at `queueWithdrawal()`, which will be at least equal but also can be higher.

This is the flow explained:

`WithdrawalPool:queueWithdrawal() -> X time passes -> StakingPool::updateStrategyRewards() so staking rewards are accrued -> WithdrawalPool::_finalizeWithdrawals()`

Now indeed the user will be withdrawing **X** shares by its actual value, yet this value will be higher than the one he specified in his first call to `PriorityPool::withdraw()`. Withdrawing more than what the user initially wanted to withdraw.

Numerical example:

- `PriorityPool::withdraw()` -> User says I want to withdraw `10 stLINK == 10 LINK`. He gets accounted let's say **10** shares.
- `X time passes` -> Rewards are accrued and the shares value is now **1.1 LINK** per share.
- A call to `_finalizeWithdrawals()` happens -> User gets accounted for withdrawal those **10** shares at a price of 1.1 **LINK** per share.
- User withdraws `10 shares * 1.1 LINK/share = 11 LINK`. Which is more than specified.

## Impact

User withdraws more than desired. This is incorrect behavior.

Furthermore those undesired withdrawn extra **stLINK** would have been earning rewards if not withdrawn, then a user is ***unwillingly stop accruing rewards on his captial***. 

Not only the user unwillingly unstakes some amount thus unwillingly stop earning rewards but also loses stake room space, a limited thus valuable resource. Which if the room was full and someone deposited after his withdrawal now the user would have to wait for more room to be freed in the pool if he wants to start receiving rewards again from that unwillingly withdrawn stake.

## Recommendations

Track the queued withdrawals in **stLINK** not shares.
