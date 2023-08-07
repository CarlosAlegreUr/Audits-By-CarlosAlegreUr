# Software Audit Report 📑

---

## **Summary 📌**

This report outlines potential exploits where well-funded attackers might tamper with the contract's storage slots, leading to higher operational costs.

---

## **Vulnerability Details 🔍**

In **`Lender.sol`**:

### Exploits by Attackers with Enough Capital

> 📘 **Notice** ℹ️: Some solutions are explained in the **Recommendations** section. They may differ in implementation so I've linked trusted implementations like OpenZeppelin contracts.

- **Infinite Pool Creation:**

  - **Problem** 🚧: The contract allows infinite pool creation, opening doors for DoS attacks via storage slot occupancy, increasing transaction costs.

- **Unregulated Loans Creation & Borrowing:**

  - **Problem** 🚧: Any address can establish a pool and increase the size of the loans array by borrowing insignificant tokens.

- **No Control over Tokens Used for Lending:**
  - **Problem** 🚧: Absence of mechanisms to control tokens lets attackers manipulate the system with only the cost of gas fees and function calls.

---

## **Impact 📈**

Redundant storage utilization can increase operational expenses.

---

## **Tools Used 🛠️**

- Manual audit.
- Slither.

---

## **Recommendations 🎯**

Considering future plans for a governance model here are some suggestions to face the problems:

- **Authorizing** the **governance to blacklist** suspicious addresses.

- **Infinite Pool Creation**:

  - Cap the number of pools an individual can establish.
    - Use a variable like `mapping(address => uint256) addressToNumOfPoolsCreated`.
    - Introduce a constant like `MAX_POOLS_PER_USER`.
    - Adjust the mapping number each time a pool is created or deleted.
    - Mark the `setPool()` function as non-reentrant, slowing the rate of pool creation and providing the governance more reaction time. For this, you can use OpenZeppelin's [ReentrancyGuard](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard).

- **Infinite Loans**:

  - Impose a cap on borrowings per address, similar to how it's addressed in the pool limits solution.
  - Note a potential reentrancy vulnerability with fake IERC20 contracts during `borrow()` calls. Again, use `ReentrancyGuard` to mitigate this risk.

- **Token Control Solutions**:
  - Enforce token whitelist/blacklist methods. For that, you could use _roles_ from OpenZeppelin's [AccessControl](https://docs.openzeppelin.com/contracts/2.x/access-control).
  - Propose a fee for pool creation or an assurance mechanism with a valuable asset, such as the native blockchain coin, reclaimable once the lending is over.

> 🚧 **Note** ⚠️: Implementing solutions to these problems requires significant code modification. A second audit is recommended post-implementation to ensure no new bugs are introduced.

---
