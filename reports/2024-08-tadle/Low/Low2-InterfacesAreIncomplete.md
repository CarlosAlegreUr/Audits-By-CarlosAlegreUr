## **Vulnerability Details 🔍**

The interface contracts are in scope. An interface must declare all external points of communication with the contract, all functions that will be called from outside the contract.

Some of them lack essential functions that are even implemented in the contracts for it to be a complete and correct interface.

See for example `ITokenManager` interface that lacks the `withdraw()` fucntion on it yet it is implemented and used by users in the contract.

---

## **Recommendations 🎯**

Revise all interfaces and complete them. Due to time constaints and having to write more reports I leave to the dev team chek this in all the interfaces.

---
