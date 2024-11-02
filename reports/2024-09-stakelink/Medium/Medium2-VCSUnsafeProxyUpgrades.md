## Vulnerability Details

All the VCS contracts, community and operator are proxies using OpenZeppelin version `4.7.0`. This kind of proxies require a storage gap variable at the end to guarantee safe upgradeability just in case the future implementation needs more state.

Every upgradeable contract that is inherited by a proxy should implement a gap in order to be safe. Yet the `Strategy` contract does not implement this gap affecting the security and safety of upgrading `CommunityVCS` and `OperatorVCS` contracts.

See there is no gap [here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/core/base/Strategy.sol#L19).
See inherited [here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/linkStaking/base/VaultControllerStrategy.sol#L48). `VaultControllerStrategy` is used by both `CommunityVCS` and `OperatorVCS`.

## Impact

Future upgrades that require more storage in the Strategy contract will corrupt the storage of the existing contracts.

## Recommendations

Add the gap variable as it is already done in other contracts of the system.
