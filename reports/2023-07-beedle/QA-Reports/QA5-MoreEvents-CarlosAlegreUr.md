## **Summary** 📌

While this report does not directly address vulnerabilities, it emphasizes best practices for the code.

---

## **Vulnerability Details** 🔍

In the `Lender.sol` contract:

- Absence of emitted events in:
  - `setLenderFee()`
  - `setBorrowerFee()`
  - `setFeeReceiver()`

Given the importance of these parameters, tracking their historical changes can be valuable for future assessments.

---

## **Impact** 📈

- **Maintainability** : Makes data more easily accessible.
- **Security** 🔒: If any strange manipulation of these values occur, the events facilitate its traceability.

---

## **Tools Used** 🛠️

- Manual audit
- Slither

---

## **Recommendations** 🎯

Introduce the following events:

```
event lenderFeeChanged(uint256 indexed newLenderFee, address indexed changedBy);
event borrowerFeeChanged(uint256 indexed newBorrowerFee, address indexed changedBy);
event feeReceiverAddressChanged(address indexed newReceiver, address indexed changedBy);
```

---
