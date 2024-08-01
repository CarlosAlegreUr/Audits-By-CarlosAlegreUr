## **Summary 📌**

An Arbitrum hard-fork can frozen the `mint()` function in `TempleGold` contract.

---

## **Vulnerability Details 🔍 && Impact 📈**

At `TempleGold` contract, the state `_mintChainId` is immutable thus set during construction.

In the `mint()` function we find the modifier `onlyArbitrum` which checks `if (block.chainid != _mintChainId) revert WrongChain();`.

Yet when blockchains have hard-forks they can decide to change their `chainid`, in the same way it happened with the well-known ethereum hard-fork.

This can also happen in Arbitrum and if so, the check on the "new" cannonical Arbitrum chain will always revert with no way of chainging it due to the immutable nature of `_mintChainId`. 

---

## **Proof Of Concept (PoC)** 👨‍💻💻

Small article on the Ethereum hard-fork: [see here](https://ethereum.org/en/history/#dao-fork)

To differentiate the 2 resulting chains on a hard-fork, the `chainid` of any of them has to change and it can perfectly be that the "cannonical" Arbitrum chain changes its own breaking the `mint()` function, the "corest" function of the whole **TempleGold** token.

---

## **Tools Used 🛠️**

- Manual review.

---

## **Recommendations 🎯**

1️⃣ Deploy slightly different `TempleGold` contracts for different chains, and only make the Arbitrum one have the `mint()` function without any `onlyArbitrum` modifier. And, in any other chain, you can for example just revert in all calls to its `mint()`.


2️⃣ If it doesn't matter giving the owner of the contract this extra power, you can also make the `_mintChainId` not immutable and create a setter the admin can call in case of hard-fork.

---
