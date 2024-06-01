### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

## **Summary 📌**

[`Adminable.sol`](https://github.com/Cyfrin/2024-05-Sablier/blob/main/v2-core/src/abstracts/Adminable.sol) contract implements a classic admin role but its trasnfer method is too weak and vulnerable aganst private key leaks or accidental transfers. As private key leaks occure more often than they should plus the contracts who use it are important and hanlde funds, this part of the code must clearly be strenghten.

As per [OpenZeppelin top 10 blockchain hacking techniques 2023](https://blog.openzeppelin.com/top-10-blockchain-hacking-techniques-of-2023) article, private key theft is in number 6. And not just any users' private key but validators keys. If stealing the key from someone as important as a validator is this high in the attack list, stealing it from a normal user who is creating airdrops is not that far fetched to be considered as Medium likelihood.

Another article showing how nowadays private key theft is a targeted attack vector:

- Different sophisticated and succesful social engineering attacks [article](https://medium.com/coinmonks/private-keys-exploit-the-most-lucrative-hack-of-2023-81390e0a29cb). 

Given the nowadays frequency and severity of private key leaks/thefts, I consider this a Medium likelihood risk with Medium impact due to admin having access to funds trhough `clawback()` function, thus a Medium risk issue.

---

## **Vulnerability Details 🔍**

The contracts that implement the weak `Adminable.sol` are:

- In `core`: `SablierV2Lockup.sol` and the contracts that inherit it, `SablierV2LockupLinear.sol`, `SablierV2LockupDynamic.sol` and `SablierV2LockupTranched.sol`.

- In `periphery`: `SablierV2MerkleLockup.sol` and the contracts that inherit it, `SablierV2MerkleLL.sol`, `SablierV2MerkleLT.sol`.

---

## Impact 📈

- In `core`: there is not much problem as the only `onlyAdmin` function which its control would be lost is the `setNFTDescriptor()`. Which only affects the `tokenURI()` view function.
  
- In `periphery`: the attacker can transfer the admin role to himself and then gain access to the `clawback()` function which could allow him to drain the funds intended to be for an airdrop campaign for example. This risk is further increased by the fact that in this case the admin could be anyone who wants to create a token distribution campaign or airdrop thus their private key hygiene won't probably be as good as it should be.

---

## **Tools Used 🛠️**

- Manual analysis.

---

## **Recommendations 🎯**

Use a 2tx step-transfer with time delay, similar to how the [`AccessControlDefaultAdminRuless.sol`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/extensions/AccessControlDefaultAdminRules.sol) from OpenZeppelin handles the `DEFAULT_ADMIN_ROLE` role, to avoid accidental transfers and protect as much as posible against compromised private keys:

- 1st step, set a new admin able to accept the role.
- 2nd, that new address sends a tx accepting the role.

Though this is not enough to properly mitigate the risks and a time wait should be added until the 2nd tx can be sent. This allows monitoring systems to detect suspicious activity and allow them to have better reaction time.

---
