# High Risk Issue: `Size` 🕵️

---

# **Normal `Liquidate` action incentives are too low due to decimals misshandling**

---

### **Explanation && Impact** 📌📈

The `liquidatorProfitCollateralToken` reward that incentivizes liquidators is always too small. We are talking about `10^(-12)` orders of magnitude smaller than it should.

This is because `liquidatorProfitCollateralToken` is determined like so:

```solidity
// 🟢1️⃣ `debtInCollateralToken` has 18 decmals but `liquidatorReward` always has 6 decimals
liquidatorProfitCollateralToken = debtInCollateralToken + liquidatorReward;

// 🟢2️⃣ This is because `liquidatorReward` is always taken from this _min_ comparison. And it compares
// `assignedCollateral - debtInCollateralToken` amount, which is in collateral decimals => 18, with the Math
// operation over a future value on credit, with 6 decimals. Clearly, the 18 decimals value will always be bigger. Around 10^(12) times bigger.
uint256 liquidatorReward = Math.min(
                assignedCollateral - debtInCollateralToken,
                Math.mulDivUp(debtPosition.futureValue, state.feeConfig.liquidationRewardPercent, PERCENT)
);
```

Liquidations of risky unhealthy positions are essential to the system and its incentives for them to happen must exit. Yet receiving just some wei's of value for that is not enough. It is true that the protocol expects up-to 5% of the credit value as extra reward for the liquidator, and this amount is so small that it is inside the 5%. But as said it is an incredibly small, `10^(-12)` orders of magnitude smaller, which can not incentivize liquidators to act as just gas cost will be bigger.

The code has already some features that tend to make positions' liqudations profitable. Like having a minimum credit amount for each loan, yet all becomes useless if you are only taking such a small fraction as the extra incentive.

---

### **Proof Of Concept (PoC)** 👨‍💻💻

Import `import "forge-std/console.sol";` in `Liquidate.sol` and add the following loggings in the `executeLiquidate()` functon [(see where in the code here)](https://github.com/code-423n4/2024-06-size/blob/main/src/libraries/actions/Liquidate.sol#L96):

```solidity
console.log("These are the 2 amounts being compared to take the min:");
console.log(assignedCollateral - debtInCollateralToken);
console.log(Math.mulDivUp(debtPosition.futureValue, state.feeConfig.liquidationRewardPercent, PERCENT));
uint256 liquidatorReward = Math.min(
            assignedCollateral - debtInCollateralToken,
            Math.mulDivUp(debtPosition.futureValue, state.feeConfig.liquidationRewardPercent, PERCENT)
);
console.log("Chosen one: ", liquidatorReward);
liquidatorProfitCollateralToken = debtInCollateralToken + liquidatorReward; 
```

Run one of the code's liquidation tests to see the logs:

```bash
forge test --match-test "test_Liquidate_example" -vvv
```

---

### **Recommended Actions** 🛠️

Use the `futureValue` converted to collateral units. This value is curiously calculated just a [few line of code above](https://github.com/code-423n4/2024-06-size/blob/main/src/libraries/actions/Liquidate.sol#L91): 

`uint256 debtInCollateralToken = state.debtTokenAmountToCollateralTokenAmount(debtPosition.futureValue);`.

```diff 
uint256 liquidatorReward = Math.min(
    assignedCollateral - debtInCollateralToken,
-    Math.mulDivUp(debtPosition.futureValue, state.feeConfig.liquidationRewardPercent, PERCENT)
+    Math.mulDivUp(/* 👁️▶️ */ debtInCollateralToken, state.feeConfig.liquidationRewardPercent, PERCENT)
);
```
