## Vulnerability Details

There is an error when using the `depositIndex` [here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/linkStaking/CommunityVCS.sol#L91).

As it can be seen the index is assigned to the `totalVaults` value. `totalVaults` in this context is meant to be the number of vaults that belong to the groups. They are just not always the same. [Here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/linkStaking/base/VaultControllerStrategy.sol#L36) it is explained what `depositIndex` actually is:

```solidity
// index of next non-group vault to receive deposits
uint64 depositIndex
```

Now non-group vaults can be more than 1, and it is true that they are placed at the end of the vaults array so it could be that this index is the index of the first non-group vault. Thus the index value is the same as the number of position of the previous element in the array, which should be the last vault in a group. Example: Array[0,1,2,3,4]. Position 5 has index 4. So if `depositIndex` is at position 5, its index indicates the amount of vaults in the array belonging to a group.

Yet this is not always true, there can be 3 non-group vaults with vault 1 being filled up, number 2 partially filled up, and number 3 empty. In this case `depositIndex` will point to 2, the second non-group vault and `totalVaults` will account for it when it should not.

## Impact

Incorrect calculation of `totalVaults` and thus `remainder` and `vaultsPerGroup` at `CommunityVCS::deposit()`. Did not have time to measure the implications further in the code, it might be that some vault groups have more deposit rooms than others and that that deposit room is greater than what Chainlink allows leading probably to future reverts of some deposits.

## Recommendations

`depositIndex` is not a reliable source that separates vault groups from non-group vaults as non-group vaults also get filled during deposits and if so `depositIndex` gets updated. See [here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/linkStaking/base/VaultControllerStrategy.sol#L262).

Use another variable or find another way to track it.
