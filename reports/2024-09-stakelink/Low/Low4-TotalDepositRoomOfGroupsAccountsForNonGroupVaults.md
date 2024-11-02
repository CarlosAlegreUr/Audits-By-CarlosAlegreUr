## Vulnerability Details

If the deposit limits on Chainlink changes, `CommunityVCS` adapts to it during the `deposit()` (see [here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/linkStaking/CommunityVCS.sol#L89)) function with this incorrect code:

```solidity
        // if vault deposit limit has changed in Chainlink staking contract, make adjustments
        if (maxDeposits > vaultMaxDeposits) {
            uint256 diff = maxDeposits - vaultMaxDeposits;
            uint256 totalVaults = globalVaultState.depositIndex;
            uint256 numVaultGroups = globalVaultState.numVaultGroups;
            uint256 vaultsPerGroup = totalVaults / numVaultGroups;
@>          👁️🔴⏬ This remainder accounts for non-group vaults.
            uint256 remainder = totalVaults % numVaultGroups;

            for (uint256 i = 0; i < numVaultGroups; ++i) {
@>          👁️🔴⏬ This `numVaults` refers to `vaultsPerGroup`.
                uint256 numVaults = vaultsPerGroup;
                if (i < remainder) {
@>                  👁️🔴⏬ Yet some `numVaults` will be added the reminder, a.k.a. vaults that are not in groups.
                    numVaults += 1;
                }
@>              👁️🔴⏬ Making those groups have accounted for them a higher depositRoom than they should. 
                vaultGroups[i].totalDepositRoom += uint128(numVaults * diff);
            }

            vaultMaxDeposits = maxDeposits;
        }
```

## Impact

Incorrect accounting of `totalDepositRoom` for some vault groups. Did not have time to explore further implications but incorrect accounting is usually the root cause of severe bugs in systems. 

## Recommendations

Do not use the `reminder` part of the logic. Those are non-group vaults which as per their own name, do not belong to any group.
