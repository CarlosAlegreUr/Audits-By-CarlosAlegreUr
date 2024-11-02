## Vulnerability Details

In the following conditions a valid depositor can get its transaction reverted with a sandwich attack and the attacker can get that deposit room for himself.

1. The valid user's deposit will fill up 1 strategy, move to the next one and fill up the next one.
2. The next strategy has less number of vaults than the previous one. (This can happen, as for example Operator strategies can remove vaults due to slashing or just Operator removals at the Chainlink pool)

If this happens, as the same `bytes data` is passed to all strategies in a deposit. See [here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/core/StakingPool.sol#L484).

Then the revert will happen [here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/linkStaking/base/VaultControllerStrategy.sol#L189) due to an out-of-bounds access error.

The `bytes data` provided are the indexes of the vaults to deposit to in a strategy, yet some strategies can have less vaults, yet the same indexes are passed which can cause an out-of-bounds access error.

## Impact

At first glance the only consequence is a reverted transaction due to incorrect logic. User can just deposit again modifying the amount such as it does not fill up too much of the second strategy and not hit an out-of-bounds index.

In a worst case scenario this can lead to another user exploiting this to hijack all the deposit room for himself with a sandwich attack. It goes as follows:

1. Attacker user sees valid user deposit.
2. Front-runs it with such a deposit amount that the next front-runned deposit will fill up the current strategy and the next one until an out-of-bounds index.
3. Now the attacker back-runs it with 2 deposits, 1 to fill up all deposit just before accessing the out-of-bounds index, and another to fill up the rest. These 2 deposits can be done in a multicall so it is just 1 tx.

## Recommendations

Query the number of vaults in a strategy and pass the `bytes data` accordingly before calling `strategy.deposit()` at `StakingPool::_depositLiquidity()`.
