## 📌 Summary

Truncation decreases accuracy in health factors' computation.

Although truncation, in this case, still makes the code's logic work as expected, improving accuracy is vital for enabling clients to make more informed decisions.

---

## 🔍 Vulnerability

In the `DSCEngine.sol` smart contract, specifically in the `_calculateHealthFactor()` method, there is a loss of precision arising from the division operation with `totalDscMinted` as denominator.

```
function _calculateHealthFactor(uint256 totalDscMinted, uint256 collateralValueInUsd)
    internal
    pure
    returns (uint256)
{
    if (totalDscMinted == 0) return type(uint256).max;
    uint256 collateralAdjustedForThreshold = (collateralValueInUsd * LIQUIDATION_THRESHOLD) / 100;
    return (collateralAdjustedForThreshold * 1e18) / totalDscMinted; //<-HERE 🔴
}
```

Let's evaluate two hypothetical yet realistic users:

1. **User 1** 🧑‍🦱

   - **Deposit**: `$343` of collateral in exchange for `$100` worth of stablecoin value.
   - Variables:
     - `collateralValueInUsd` = 343.
     - `totalDscMinted` = 100
   - Real health factor = (343 \* 0.5)/100 = 1.715

2. **User 2** 🙆
   - **Deposit**: `$201` of collateral in exchange for `$100` worth of stablecoin value.
   - Variables:
     - `collateralValueInUsd` = 201.
     - `totalDscMinted` = 100
   - Real health factor = (201 \* 0.5)/100 = 1.005

Yet the health factors are different, the value returned by `_calculateHealthFactor()` is 1 for both users. This discrepancy arises because, while the code's division values are adjusted for decimals, the numerator isn't multiplied by a significant factor to treat the result as a real number instead of an integer.

> 📘 **Notice** ℹ️: Considering the existence of a public function for computing health factors in the contract, it's reasonable to assume the protocol isn't expecting clients to perform off-chain health factor computations.

---

## 📈 Impact

This imprecision, while not causing logical errors in the protocol, can introduce confusion for users leading to misconceptions about other user positions.

---

## 🛠️ Tools Used

- Manual audit
- Slither

---

## 🎯 Recommendations

To avoid truncation and get accurate health factor calculations:

1. Introduce a new constant named `HEALTH_FACTOR_PRECISION`. It's value should be `10^(desired number of decimals)`.
2. Adjust the `_calculateHealthFactor()` accordingly.
3. Add a getter function so clients can consult the new constant.

   <details> <summary> Some adjustments examples 🏗️ </summary>
   The adjusted code snippet would look like:

   ```
   // New constant definition 🟢
   uint256 private constant HEALTH_FACTOR_PRECISION = 1e2

   // contract's code...

   function _calculateHealthFactor(uint256 totalDscMinted, uint256 collateralValueInUsd)
       internal
       pure
       returns (uint256)
   {
       if (totalDscMinted == 0) return type(uint256).max;
       uint256 collateralAdjustedForThreshold = (collateralValueInUsd * LIQUIDATION_THRESHOLD) / 100;

       // Modification here 🟢
       return (collateralAdjustedForThreshold * 1e18 * HEALTH_FACTOR_PRECISION) / totalDscMinted;
   }

   // contract's code...

   // Additional getter 🟢
   function getHealthFactorPrecision() external view returns (address) {
       return HEALTH_FACTOR_PRECISION;
   }
   ```

   > 🚧 **Note** ⚠️: This code has not been tested, it's meant to serve as a reference.

   </details>

4. Update the contract's logic that deals with health factors' values to account for this new constant.

> 🚧 **Note** ⚠️: It's crucial for users to be aware of the number of decimals in a health factor. Examples on how to consult certain information should be added in the final docs of the contract.
