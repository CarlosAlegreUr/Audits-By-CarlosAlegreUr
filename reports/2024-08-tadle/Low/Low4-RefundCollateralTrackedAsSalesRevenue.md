## **Vulnerability Details 🔍 && Impact 📈**

At `DeliveryPlace.sol` at `settleAskMaker()`, `makerRefundAmount` is uesed in `addTokenBalance()` with the type `TokenBalanceType.SalesRevenue`. It should be `TokenBalanceType.MakerRefund`.

Coudnt find any negative consequence on the users as `SalesRevenue` and `MakerRefund` both have the same collateral units. Impact is incorrect state handling. 

---

## **Recommendations 🎯**

Track it as `MakerRefund`, the line is [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/DeliveryPlace.sol#L302)

---