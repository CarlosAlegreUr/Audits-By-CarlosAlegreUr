## Vulnerability Details

You can only withdraw from a strategie's vaults that belong to the current unbonded gruop (`curUnbondedVaultGroup`).

Not all strategies will have the same vaults inedexes at the same time belonging to the same unbonded group.

One strategy might have `vaults = [0, ..., 5]` at the `curUnbondedVaultGroup`. And the next one might have `vaults = [0, ..., 3]`.

Yet for both, if a withdrawal is large enough, the same indexes will be passed. Resulting in an `error InvalidVaultIds()` when the code starts withdrawing from the second strategy looping the provided indexes. See loop [here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/linkStaking/base/VaultControllerStrategy.sol#L126), see belonging to `curUnbondedVaultGroup` check [here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/linkStaking/base/VaultControllerStrategy.sol#L128). When index 4 is used in the second strategy, as it does not belong to current group it reverts.

Here you can see that the same `bytes data` is passed to all strategies a withdrawal involves. See [here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/core/StakingPool.sol#L511).

## Recommendations

To avoid annoying reverts to users query the indexes of the current unbonded vaults before calling `strategy.withdraw()` at `StakingPool::_withdrawLiquidity()`.
