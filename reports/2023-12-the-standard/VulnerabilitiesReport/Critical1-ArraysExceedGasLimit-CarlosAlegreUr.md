### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

### **DOS due to the way staking positions are tracked** 🔢

## **Summary 📌**

The staking part of the system loops over potentially too big arrays. A malicious actor can easily overinflate this arrays to deny service of all functions that use them.

This is easier than expected due to the expensive operations done in these function in terms of gas. Having big arrays would quickly hit the block gas limit and effectively create a DOS (denial of service).

---

## **Vulnerability Details 🔍 && Impact 📈**

The 3 arrays that cause this problem are:

- `holders` in `LiquidationPool`.
- `pendingStakes` in `LiquidationPool`.
- `tokenIds` in `SmartVaultIndex`. 

> ℹ️ **Note** 📘: Notice the index contract is out of scope but it's used in the system and can potentially revert `_afterTokenTransfer()` in `SmartVaultManager` which is in-scope. `vaults()` in `SmartVaultManager` also suffers from this issue but this function is acknowledge as a known issue.

These arrays are potentially very big due to the `.push()` operations that are executed when users: `inreasePosition()` at `LiquidationPool` or `mint()` vaults.

| Contract                 | Functions Vulnerable to DoS  |
| ------------------------ | ---------------------------- |
| `LiquidationPool`        | `getTstTotal()`              |
|                          | `position()`                 |
|                          | `distributeFees()`           |
|                          | `deletePendingStake()`       |
|                          | `consolidatePendingStakes()` |
|                          | `addUniqueHolder()`          |
|                          | `increasePosition()`         |
|                          | `distributeAssets()`         |
| `LiquidationPoolManager` | `runLiquidation()`           |
|                          | `distributeFees()`           |
| `SmartVaultManager`      | `_afterTokenTransfer()`      |


All these are functions crutial to the systems' functionality and would render the system unusable while leaving funds stuck there forever. Given the fact that the [provdied TST contract](https://arbiscan.io/token/0xf5a27e55c748bcddbfea5477cb9ae924f0f7fd2e) has already 111 holders. Its reasonable to assume that the amount of stakers will be big enough to, even not maliciously, cause a DOS.

That is why this system's logic might not be flawed but it will be unable to satisfy client needs when demand scales or just starts existing. This a ***`CRITICAL`*** point to fix.

Furthermore, essential operations like `runLiquidation()` have lots of logic inside and sometimes they even iterate more than once over this arrays which only makes the gas limit problem more easy to appear. Even if just 1 single address sends lots of pending stakes with low economical value a DOS attack could start to `runLiquidation()`, one of the most essential functions to the system.

---

## **Tools Used 🛠️**

- Manual audit.

---

## **Recommendations 🎯**

1️⃣ Change the staking prizes distribution approach to a more gas efficient one.
For example:

- Rewards are updated globaly. In your case you can leverage the fact that you have a whitelisted assets list to just create a mapping like: `mapping(address => uint256) rewards;`.
Every time rewards need to be updated now you just have to loop trhough the whitelisted tokens, which is an array with length managed by the team which can make sure to never cause DOS.

- Then, each user, when claiming rewards will just get a portion of the rewards of each asset based on their % TST owned in the pool for example. You can add the distribution logic of purchased assets at discount that is now inside the `distributeAssets()` logic within `claimRewards()`. Now it would be to each holders responsibility to claim their rewards whenever they see fit. Avoinding this way the DOS loop over the `holders` array.

2️⃣ To solve the problem with the pending stakes add a new parameter to the `consolidatePendingStakes()` function. This parameter will be something like: `uint256 indexFrom`. And instead of looping trhough the beggining till the end you will only consolidate an amount of stakes which can be exeuted by the solidity's `gasleft()`. You should test how much gas is consumed by each consolidation and create on-code logic with the `gasLeft()` function to dynamically use the indexes to consolidate. 

This still creates the risk of the pending stakes towards the begining of the arrey never to be reached, for that best I can think of is add another parameter (`uint256 indexTo`) for range-wise consolidation that can be activated by anyone so the own staker can call it himself and consolidate his own stake after the cosolidation period has passed.

3️⃣ For the indexing NFTs problem checkout the [ERC721Enumerable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/extensions/ERC721Enumerable.sol) extension from OpenZeppelin.

---