# QA/Low Risk Report for `Panoptic` 🕵️

### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

---

## Summary 📌

Apart from the 6 different **QA** aspects, **_`3 low-risk`_** issues were found.

---

## Index 🗂️

- [QA]() 🧩📝
  - [1️⃣ Refactor](#refactor-1️⃣)
  - [2️⃣ Modularity](#modularity-2️⃣)
  - [3️⃣ Comment correctness](#comment-correctness-3️⃣)
  - [4️⃣ Consistency & readability](#consistency--readability-4️⃣)
  - [5️⃣ Naming](#naming-5️⃣)
  - [6️⃣ Code & Repo Navigability](#code--repo-navigability-6️⃣)
- [Low Risk 🔅](#low-risk-🔅)
  - [Revert if on deposits shares to mint = 0](#low-risk-1️⃣-🔅)
  - [Solidity pragmas should be restrictive](#low-risk-2️⃣-🔅)
  - [Owner system of factory contract is risky](#low-risk-3️⃣-🔅)
 
---

# QA 🧩📝

## **1️⃣ `Refactor`** 🧩

### **1️⃣.1️⃣ Define errors an events in interfaces**

It's a good practice to define errors and events in interfaces, and add the functions docs for the interface there. Allow for developers for faster understanding of the code and the functions that are being used. Also encapsulates better the logic of the contracts and makes the contract file smaller and more comfortable to work with.

For example with the `PanopticPool.sol` a `IPanopticPool.sol` interface should be made with all the **external** and **public** functions and their docs. In a glimpse a developer would see that the main actions users do in the pool are: `mintOptions()`, `burnOptions()`, `forceExercise()`, `settleLongPremium()` or `pokeMedian()`.

Apply this interface pattern to all contracts.

### **1️⃣.2️⃣ Private functions for computing long mapping keys**

When keys are hashes of a lot of parameters create a private function that returns the hash is more modularized and clean:

```solidity
// Code would go from this:
 bytes32 positionKey_from = keccak256(
                abi.encodePacked(
                address(univ3pool), from, id.tokenType(leg), liquidityChunk.tickLower(), liquidityChunk.tickUpper())
            );

 // To something like this:
 bytes32 positionKey_from = _getPositionKey(univ3pool, from, id.tokenType(leg), liquidityChunk.tickLower(), liquidityChunk.tickUpper());
```

This also applies to other long keys the code has like `chunkKey`.

### **1️⃣.3️⃣ 2 internal view function should be external or public view**

In `PanopticPool.sol`, the functions `getUniV3TWAP()` and `_checkSolvencyAtTick()` should be exetrnal so traders can better assess if they are liquidatable by `liquidate()` like so:

```solidity
_checkSolvencyAtTick(getUniV3TWAP());
```

## **2️⃣ `Modularity`** 🧩

- This block of code implementing an `if else if` statement is repeated twice in `CollateralTracker.sol`. [Here]() in `takeCommissionAndAddData()` and [here]() in `exercise()` function. This could be modularized into a separate private function called `_mintBurnPayingAmountShares()`.

- In the `liquidate()` function, from [here](https://github.com/code-423n4/2024-04-panoptic/blob/main/contracts/PanopticPool.sol#L1038) until the end of the brackets [here](https://github.com/code-423n4/2024-04-panoptic/blob/main/contracts/PanopticPool.sol#L1067). Could be replaced by a call to `_checkSolvencyAtTick()` internal function. I tried it and tests still pass. Like so:
 
```solidity
_checkSolvencyAtTick(liquidatee, positionIdList, currentTick, twapTick, NO_BUFFER);
```

## **3️⃣ `Comment correctness`** 🧩

- Comments in `CollateralTracker.sol` for `BUYER_COLLATERAL_RATIO` and `SELLER_COLLATERAL_RATIO` are switched. [See them](https://github.com/code-423n4/2024-04-panoptic/blob/main/contracts/CollateralTracker.sol#L139).

- In [this comment](https://github.com/code-423n4/2024-04-panoptic/blob/main/contracts/PanopticPool.sol#L264) it is said that `abi.encodePakced()` is used for the hash of the XORs. But in the code `abi.encode()` is used. [See here](https://github.com/code-423n4/2024-04-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L103).

- It returns liquidability threshold, not insolvancy threshold. Usage is correct though, only thing wrong is the docs. [Here](https://github.com/code-423n4/2024-04-panoptic/blob/main/contracts/PanopticPool.sol#L1338).

- [Here](https://github.com/code-423n4/2024-04-panoptic/blob/main/contracts/PanopticPool.sol#L1481) the variable is not actually `totalLiquidity`, the value which holds is actually `removedLiquidity`. Change name to `removedLiquidity`.

## **4️⃣ `Consistency & readability`** 🧩

- Use `i` as the iterator name in [this foor loop](https://github.com/code-423n4/2024-04-panoptic/blob/main/contracts/PanopticPool.sol#L441) for consistency.

- In the `LeftRight.sol` type usually token0 related values are in the right side (least-significant-bits) and token1 related values are in the left side (most-significant-bits). When operating with these values for example in the [return values](https://github.com/code-423n4/2024-04-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1522) of `SFPM.getAccountPremium()` keep the left-right order for consistency as it will be easier to not mess up during development. Like `return(LeftRight.left(), LeftRight.right())`, return the left value as the first arg and the right value as the second arg. It is easier to read and understand and consistent how values are visually arranged in `LeftRight` type.

- [Here](https://github.com/code-423n4/2024-04-panoptic/blob/main/contracts/PanopticPool.sol#L795) in `_burnAllOptionsFrom()`, write its inputs in the same order as `_burnOptions()` for consistency and faster readability.
  

## **5️⃣ `Naming`** 🧩

- `MEDIAN_UPDATE_PERIOD` is better than `MEDIAN_PERIOD`. [Here](https://github.com/code-423n4/2024-04-panoptic/blob/main/contracts/PanopticPool.sol#L136)
 
- `s_settledPremiaTokens` is better than `s_settledTokens`. [Here](https://github.com/code-423n4/2024-04-panoptic/blob/main/contracts/PanopticPool.sol#L251).

- `_poolUtilization()` could be more readable as `_uniswapPoolUtilization()`. [Here](https://github.com/code-423n4/2024-04-panoptic/blob/main/contracts/CollateralTracker.sol#L741).

- `_createPositionInAMM()` is better fit with `_modifyPositionInAMM()`. As it not just creates positions but also burns them. [Here](https://github.com/code-423n4/2024-04-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L863)

## **6️⃣ `Code & Repo Navigability`** 🧩

### 6️⃣.1️⃣ Comments navigability.

- Use `/** */` for natspec longer than 1 line, some famous editors like **VSCode** allow to collapse it, making in practice a more navigable code.

- In `libraries/Errors.sol` classify them with comments according to the kind of errors, like:

```solidity 
///////////////////////////////
//// Access Control Errors 
///////////////////////////////

errors....

///////////////////////////////
//// Math errors
///////////////////////////////
```

### 6️⃣.1️⃣ Repo navigability.

- Create more directories to better encapsulate the different parts of the system.

For example inside `/contracts/libraries`, classify libraries according to their functionality, like `/contracts/libraries/math`, `/contracts/libraries/panopticUtils` and `/contracts/libraries/uniswapInteractions`.

- On the tests, instead of a really big test file for each contract. Create a directory for each contract and inside it create a file for each functionality tested. This way the tests are more modular and easier to navigate.

Examples: `/tests/CollateralTracker/deposit.t.sol`, `/tests/CollateralTracker/withdraw.t.sol`, `/tests/PanopticPool/forceExercise.t.sol`, `tests/PanopticPool/mintOption.t.sol`.

---

---

# **Low Risk** 🔅

---

## Low Risk 1️⃣ 🔅

### `Revert if on deposits shares to mint = 0`

---

### Explanation && Impact 📌📈

In `CollateralTracker.sol`, even though measures like virtual shares have been taken for the classic first depositor problem with ERC4626 vaults, an extra check should be added for extra security in case a well-funded individual tries to attack someone.

The check is the following:

```solidity
  function deposit(uint256 assets, address receiver) external returns (uint256 shares) {
        if (assets > type(uint104).max) revert Errors.DepositTooLarge();

        shares = previewDeposit(assets);
        if(shares == 0) {revert;} // 🟢 NEW
      // rest of code...
  }
```

---

### **Recommended Action** 🛠️

Add the check shown in the code snippet above to avoid well-funded individuals attacking first depositors.

---

## Low Risk 2️⃣ 🔅

### `Solidity pragmas should be restrictive`

---

### Explanation && Impact 📌📈

The `pragma solidity ^X.X.X` or any other range-wise pragma should be avoided in favor of more restrictive ones, like `pragma solidity X.X.X` to avoid or narrow down risks with any potential issues with future compiler versions or bugs.

---

### **Recommended Action** 🛠️

Use restrictive pragmas.

---

## Low Risk 3️⃣ 🔅

### `Owner system of factory contract is risky`

---

### Explanation && Impact 📌📈

1️⃣ In the event of compromised keys, the owner system of the factory contract is risky. A timelock should be added to the owner system to avoid and have reaction time against any potential issues. And for top security, add a special address owned by the team that can be transfered the owner role with no time delays so in case of leakage the team can retake the control more easily.

2️⃣ Also a 2tx process should be added to avoid any accidental transfers to unintended addresses. Process goes like this, in 1 tx a new potential owner is set, and that new potential owner has to accept the ownership by sending a second accepting tx. 

### **Recommended Actions** 🛠️

Add the described timelock to the owner system and a 2tx process to avoid any potential issues.

Because the ultimate owner is meant to finally be `address(0)`, when implementing the 2tx transfer process add an `if` statement to avoid the 2tx process if the `newOwner` is `address(0)`. Yet still the timelock should work just in case an accidental or malicious transfer to `addres(0)` happens before the expected time and so it can be detected and reversed on time.

---

---
