## Vulnerability Details

At `OperatorVCS` during `queueVaultRemoval()` the group the vault being queued belongs to is wrongly calculated.

Group is calculated like so (see [here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/linkStaking/OperatorVCS.sol#L288)):

```solidity
uint256 group = _index % globalVaultState.numVaultGroups;
```

It should be `_index / vaultsPerGroup` not module.

Imagine each group has 2 vaults and we are queuing vault 10.

Vaults = [0,1,2,3,4,5,6,7,8,9,10,11]
VaultsGroup = [ [0,1], [2,3], [4,5], [6,7], [8,9], [10,11] ]

Vault 10 is at group **6**.

Code does: `_index % globalVaultState.numVaultGroups == 10 % 6 == 4`

The proper calculation should be `_index / vaultsPerGroup` to know in which group on the `vaultGroups` array the vault is.

## Impact

The following call to: `fundFlowController.updateOperatorVaultGroupAccounting(groups);`

Will not update the correct group. This is really important to be done correctly as once a vault is queued for removal it can't be queued again (due to this check [here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/linkStaking/OperatorVCS.sol#L281)) and the accounting will be broken until another vault from that very same group that should have been updated gets removed.

## Recommendations

Use `_index / vaultsPerGroup` to determine the belonging group of the vault.
