### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

## Summary

The protocol uses OpenZeppelin `"@openzeppelin/contracts-upgradeable": "^4.9.0"`. These contracts are deployed behind proxies in order to be upgradeable, for that there is a security measure of adding a `__gap` variable in order not to have corrupted storage during upgrades and in order to allow contracts to have more state variables in case they need it in the upgrades. The problem is that the current inheritance structure of the system contracts does not guarante upgrade safe contracts.

## Vulnerability Detail

All contracts of the system are unsafe to upgrade due to inheriting non-upgrade safe contracts or not having a `__gap` variable. 

This is the inheritance structure of the system contracts:

#### Legend 🔑

- 🔴 color -> means that the contract should have a `__gap` BUT it doesn't
- 🟢 color -> means that the contract should have a `__gap` AND it does
- ⚪ color -> means the contract needs no state variables thus does not require a gap

> 🚧 **Note** ⚠️ If image does not display you can see it [here](https://github.com/CarlosAlegreUr/Audits-By-CarlosAlegreUr/blob/main/reports/2024-05-midas/midas-gap-inheritance.png).

<img src="https://raw.githubusercontent.com/CarlosAlegreUr/Audits-By-CarlosAlegreUr/main/reports/2024-05-midas/midas-gap-inheritance.png" alt="inheritance-gap-incosistencies">

As you can see in the diagram:

- `DepositVault` & `RedeemVault` inherit from 2 unsafe contracts: `ManagableVault` & `WithMidassAccessControl`. Notice that these 2 contrats are also interconnected.

- `mTBILL` ERC20 contract & `DataFeed` contract inherit from the unsafe `WithMidassAccessControl` too.

- `MidasAccessControl` does not inherit from any unsafe contract yet it does not have a `__gap` variable thus it is not upgrade safe.

> 🚧 **Note** ⚠️ There is also another risk. The gaps that have been added to the contracts are not consistent with OpenZeppelin standard, which is a total of 50 storage slots per contract. See [here](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps). This makes upgradeability more nuanced and prone to errors.

## Impact

All contracts in the system can be corrupted during upgrading, thus causing the state to be broken and unconsistent. This means, for example, important state variables (among lots others) like `tokensReceiver` in `ManagerVault` can get corrupted.

## Code Snippet

- [Here](https://github.com/sherlock-audit/2024-05-midas/blob/main/midas-contracts/contracts/DepositVault.sol#L26) you can see that `DepositVault` does inherit from `ManagableVault` and [here](https://github.com/sherlock-audit/2024-05-midas/blob/main/midas-contracts/contracts/abstract/ManageableVault.sol#L52) you can see that `ManagableVault` does not have a `__gap` variable at the end of its state declaration. The other contracts in red on the image above have the same issue. I will just link the most influential of all, see `WithMidasAccessControl.sol` [here](https://github.com/sherlock-audit/2024-05-midas/blob/main/midas-contracts/contracts/access/WithMidasAccessControl.sol#L24).

- A clear example different than 50 total slots used in `RedemptionVault`, [see here](https://github.com/sherlock-audit/2024-05-midas/blob/main/midas-contracts/contracts/RedemptionVault.sol#L40).

- See version of OZ contracts used [here](https://github.com/sherlock-audit/2024-05-midas/blob/main/midas-contracts/package.json#L55) on the package.json.

Other codebases on Sherlock contests with complex unsafe upgrade paths are deemed as valid Meidum:

- [Click to see one.](https://github.com/sherlock-audit/2022-09-notional-judging/issues/64)

## Tool used

Manual Review

## Recommendation

Add the `__gap` state variable at the end of each contract in red on the iamge above. And make it so the total storage slots used by each contracts is 50.
