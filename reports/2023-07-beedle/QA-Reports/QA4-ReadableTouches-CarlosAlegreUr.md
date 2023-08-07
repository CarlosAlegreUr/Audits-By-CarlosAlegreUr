## Summary 📌

In this report, we can find 2 QA-related improvements that enhance the readability of the code through appropriate and consistent naming and labeling.

## Vulnerability Details 🔍

### 1️⃣ **Lack of named constants for readability**:

There are some hardcoded constants throughout the code. Naming them would make the code more readable.

For example:

```
// Instead of 🔴
if (_fee > 5000) revert FeeTooHigh();

// Use 🟢
uint256 public constant MAX_LENDER_FEE = 5000;
(code...)
if (_fee > MAX_LENDER_FEE) revert FeeTooHigh();
```

> 📘 **Notice** ℹ️: Details about all the hardcoded constants and name suggestions can be found in the **Recommendations** section.

### 2️⃣ Private visibility is more fitting for these functions 👁️

If the `Lender.sol` contract is NOT intended to serve as a base contract for inheritance, it's advised to convert certain or all `internal` functions to `private`. Specifically, I'm referring to the `_updatePoolPrice()` and `_calculateInterest()` functions.

Typically, the `internal` visibility is used in anticipation of contract inheritance. Hence, if `Lender.sol` will not serve as a base, these functions would be more appropriately labeled as `private`.

## Impact 📈

The changes will lead to improved code readability and a more clear understanding of each function's purpose.

Furthermore, after applying these changes, client tests still pass successfully.

## Tools Used 🛠️

- Manual audit.

## Recommendations 🎯

Below are the proposed names for the hardcoded constants, along with the lines they appear on:

| Line(s)                 | Value      | Proposed Name        |
| ----------------------- | ---------- | -------------------- |
| 85                      | 5000       | MAX_LENDER_FEE       |
| 93                      | 500        | MAX_BORROWER_FEE     |
| 246, 384, 618           | 10 \*\* 18 | TOKEN_DECIMALS       |
| 265, 561, 650, 724, 725 | 10000      | BASIS_POINTS_DIVISOR |
