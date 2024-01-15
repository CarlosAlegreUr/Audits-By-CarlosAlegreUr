### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

### 2 tx setters && delay are needed for more robust and trust-worthy system 🔢

## **Summary 📌**

Multiple crucial system parameters setter functions are a 1 tx process, `onlyOnwer` and with no time delay. This can be dangerous as mistakes can be done plus users don't have time to react if they don't agree or trust the new settings or owner of the protocol.

---

## **Vulnerability Details 🔍 && Impact 📈**

#### 1️⃣ `Setters for external contracts addresses 📜`
The system interacts with a bunch of external contracts that are trusted but which its address could be changed anytime by the owner. In case of corruption users have no time to respond and in case of setting accidentally a wrong address damage could be done to users operating until a proper address is set. Examples of these are in `SmartVaultManager` The `setLiquidatorAddress()`,`setSwapRouter2()`,
and `setWethAddress()` functions among multiple others.

#### 2️⃣ `Setters for crucial system parameters ⚙️`

The setters that modify the fees by the protocol when minting, burning, swaping or earning staking rewards have also this problem. In case of a mistake or corruption users have no time to respond to, for example, an abusive fee. Examples of these are in `SmartVaultManager` the `setBurnFeeRate()`, `setSwapFeeRate()`, `setPoolFeePercentage()`.

#### 3️⃣ `Ownership transfers 🕴️`

The ownership of some contracts like `LiquidationPoolManager` uses `Ownable` by OpenZeppelin. Accidental transfers can happen and this would be horrific as the owner in this system has a lot of power. A little example of it among lots of them could be that he sets a burn rate of `100%` so anyone who wants to retrieve their collateral cant do it, and even if they do they will lose all the money. Please change the model to the other `Ownable2Step` contract which OpenZeppelin also provides. On top of that a time delay in ownerwship transfer is recommended to allow users to decide if they trust the new owner.

---

## **Tools Used 🛠️**

- Manual audit.

---

## **Recommendations 🎯**

To prevent accidental damage implement a 2 tx process for the setters and use `Ownable2Step` contract for ownership.

New setters workwlows:
1. A tx that announces what will be the next parameter.
2. A tx where the parameter is set after checking (off-chain or on-chain) if the value is the expected one.

And to increase trust worthyness with clients implement a time delay in which once a parameter is set it will actually start taking action.

> 🚧 **Note** ⚠️ : Notice if the system is found to be vulnerable to something that would require a parameter change to be fixed, the time delay can cause more harm than good. It is all about what the dev team wants to prioritize, corruption risks or quicker fixes towards these kinds of problems. Regardless of that, at least, the `Ownable2Step` contract should be used, an accidental transfer is too dangerous.

---
