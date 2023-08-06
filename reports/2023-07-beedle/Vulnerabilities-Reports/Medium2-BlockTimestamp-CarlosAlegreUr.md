## **Summary 📌**

Using `block.timestamp` within `Lender.sol` presents potential **exploits** by block validators.

---

## **Vulnerability Details 🔍**

### **Manipulation by Block Validators Using `block.timestamp`**

Block validators possess the ability to influence the `block.timestamp` for the blocks they validate. Such a capability can be weaponized to favor validators or to hinder genuine users, leading to unexpected system behaviors such as **transaction reversals**. As a result, **legitimate users might face losses**.

> 📘 **block.timestamp Insight ℹ️**: block.timestamp's value will always be larger than its preceding block and smaller than its subsequent block. However, the exact time difference can vary. This mechanism ensures the **synchronized operation** of the blockchain.

### **1. Interest Rate Manipulation by Validators 📊**

Inside the `_calculateInterest()` function, the code:

```
uint256 timeElapsed = block.timestamp - l.startTimestamp;
```

**can be exploited**. Validators running a pool might exploit this part of the code when the function is called through the `repay()` method. They could artificially inflate the timestamp to accrue added interest. Conversely, they could be incentivized to set a lower `l.startTimestamp` to enjoy slight benefits during loan repayments.

While this **vulnerability** may offer minor immediate financial gains and affect only a few addresses, it still presents a **systematic inconsistency**. Addressing this discrepancy or warning users about it will likely create greater confidence among users.

Additionally, **validators could** manipulate the interest computations for other users or **obstruct** their **repayments**. Such interference could lead to transaction reversals, especially if the manipulated interest exceeds expected values.

This issue is present in other functions like `refinance()`, `buyLoan()`, and `zapBuyLoan()`. Regarding `refinance()`, its implications are similar to those explained for `repay()`, leading to slight variations in interest rates.

### **2. Auction Outcome Manipulation 🛍️**

In the functions `buyLoan()` and `zapBuyLoan()`, validators can influence **auction outcomes**. For example, a user executing a `buyLoan()` or `zapBuyLoan()` just before an auction ends might be abused by this exploit:

```
if (block.timestamp > loan.auctionStartTimestamp + loan.auctionLength)
```

This part of the code could be exploited to end the auction a few seconds or minutes before the expected time, thus resulting in the legitimate user's transaction being reverted and the auction being closed without any buyer when it should have.

---

## **Impact 📈**

Such behaviors, even if favouring only a fraction of users, **deviate from the protocol's intended operations**.

---

## **Tools Used 🛠️**

- Manual audit.
- Slither.
- Visual Studio Solidity Visual Developer: Function Dependencies graph.

---

## **Recommendations 🎯**

- Implementing a **commitment-reveal scheme** should be considered. While this might introduce additional complexity, it offers a robust defense against timestamp manipulation.

  > 📘 **Note** ℹ️: `Lender.sol` has another vulnerability, front-running, this vulnerability can be solved via commitment-reveal scheme too and I've sent another report about it with more details.

- Future **governance** of the protocol should actively **supervise block validators**. **Blacklisting** misbehaving validators can motivate ethical conduct. However, the dynamic creation of new addresses by validators remains a concern.

- Since timestamps also dictate the beginning of an auction through `startAuction()`, it's advised for **users and developers to use only the on-chain timestamp** value just after their transaction block is validated, rather than relying on the timestamp of when they sent the transaction.

In keeping with the protocol's goal to stay Oracle-less, these are my primary recommendations.

>  🚧 **Note** ⚠️: These changes on the code would be complex, so another audit is needed after it's implementation.