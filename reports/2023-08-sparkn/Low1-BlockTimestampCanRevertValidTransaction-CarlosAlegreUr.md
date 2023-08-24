<hr/>

## **Summary 📌**

In the **`ProxyFactory.sol`** contract, utilizing `block.timestamp` — specifically when creating contests with brief durations (**duration <= block_creation_time_in_blockchain**) — might result in a **valid transactions being reverted**.

---

## **Vulnerability Details 🔍**

Occasionally, due to either malicious intent or not, validators might adjust the `block.timestamp` to a value slightly preceding the `closeTime`.

This leads to the the condition in _`setContest()`_ (line 110) being **true** and executing an unintended revert:

```solidity
        if (closeTime > block.timestamp + MAX_CONTEST_PERIOD || closeTime < block.timestamp) {
            revert ProxyFactory__CloseTimeNotInRange();
        }
```

This malfunction occurs if:

- The provided timestamp was **valid**: `closeTime > previousBlockTimestamp`
- The intended duration for the contest was an expected small one.

---

## **Impact 📈**

The repercussions are **little**, some lost time and a small amount of gas wasted due to the unexpected reverts.

---

## **Tools Used 🛠️**

- Manual audit.

---

## **Recommendations 🎯**

Two potential fixes are suggested:

> 🚧 **Note** ⚠️: Given that this project is said to be deployed across diverse EVM-compatible blockchains, consider their different block creation intervals across these platforms when fixing this issue.

- **1️⃣ Abandon the concept of immediate contest conclusions** and set a minimum duration equivalent to the average block creation interval on the targeted blockchain.

- **2️⃣ Evaluate, when `closeTime` is inferior to `block.timestamp`, if the time gap** between `block.timestamp` and `closeTime` falls within X seconds (where X denotes the block creation interval). This ensures the reliability of genuine situations and prevents a contest from starting too early. At its extreme, a contest could kick off X seconds prior to real time, which shouldn't disrupt other features.

    <details><summary> Code example 💻 </summary>
    
    > 🚧 **Note** ⚠️: This provided code has not undergone testing and is meant to serve as a guideline.

  ```solidity
    function setContest(/*args*/) public onlyOwner
    {
      // More code...

        if (closeTime > block.timestamp + MAX_CONTEST_PERIOD) {
            revert ProxyFactory__CloseTimeNotInRange();
        }

        // 🟢 Here is the new condition 🟢
        if (closeTime < block.timestamp && block.timestamp - closeTime > AVERAGE_BLOCK_CREATION_TIME) {
            revert ProxyFactory__CloseTimeNotInRange();
        }

      // More code...
    }
  ```

    </details>

For its simplicity, I particularly recommend the **solution 1️⃣**.
