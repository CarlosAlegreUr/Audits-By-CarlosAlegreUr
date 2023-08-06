## Summary 📌

Reentrancies are widespread throughout various functions in `Lender.sol`.

> 📘 **Metrics** ℹ️: 10 out of 21 functions (47.6%) are susceptible to reentrancy which leads to attacks.

These vulnerabilities can lead to a lot of undesirable behaviors, from minor glitches to significant financial losses for users. This report highlights some identified scenarios, but other undiscovered exploits might still exist.

---

## Vulnerability Details 🔍

**Reentrancies in the contract** can have various implications:

- Some might just trigger **duplicate event emissions**. 🔄
- Some can cause **unnecessary storage consumption**. 📦
- The most severe can allow **malicious actors** to scam funds from valid users. 🚨

The root cause of these issues is the unregulated external calls to **ERC20 contracts**. Educating users about reviewing token contracts before transactions can mitigate risks. However, relying on user diligence alone is not realistic if we expect mass adoption.

For **widespread adoption**, building inherently secure applications is crucial. Relying on users to be consistently vigilant exposes them to potential exploitation, especially by malicious **ERC20 tokens**.

### Severity Descriptions 📖

| Symbol | Explanation                                                                         |
| ------ | ----------------------------------------------------------------------------------- |
| 🔴     | Critical risk: Potential loss of user funds.                                        |
| 🟡     | Moderate risk: Risk of unwanted behaviors and exploit of storage consumption.       |
| ⚪     | Observational: Reentrancy detected but no immediate harmful effect identified, yet. |

---

### Functions with reentrancy list: Examples and explanations 📖

<details> <summary> repay() 🔴 </summary>

### _**`repay()`**_

**Scenario**:

- Pool configuration: Debt represented by **Scammers token (STKN)**, Collateral could be any valuable token like **WBTC**.
- Malicious Borrower (MB): This is just a disguise. In reality, it's another address owned by the pool's lender.
- There are legitimate users whom the scammers have persuaded to use their **STKN**.

**Steps**:

1️⃣. MB borrows a certain amount of **STKN**, providing **300 WBTC** as collateral.

2️⃣. A legitimate user borrows **STKN**, depositing **700 WBTC**.

3️⃣. The malicious borrower (MB) then calls `repay()`. However, the **STKN** contract contains reentrant code which triggers `repay()` again, causing it to execute twice.

Assuming MB uses the same loanId for both calls to `repay()`:

- The first call acts as expected, returning **300 WBTC** to MB.
- The second call, however, extracts another **300 WBTC** from the pool. This is because the pool, at that moment, has a total of **1000 WBTC** and believes it has sufficient balance to return **300 WBTC**. This is despite the fact that those **1000 WBTC** were deposited by different users.

Visualization of the second call to `repay()` with inline explanations. Read first the comment marked with 1️⃣❗❗, then go to the top of the function again and read the comments in execution order:

> 🚧 **Note** ⚠️: If you are not familiar on how a reentrancy attacker contract looks like here is an example: [Solidity by Example, Reentrancy](https://solidity-by-example.org/hacks/re-entrancy/)

```
function repay(uint256[] calldata loanIds) public {
    for (uint256 i = 0; i < loanIds.length; i++) {
        // 2️⃣ (start reading on the emoji below saying 1 👁️)
        // Reentrancy has been called before delete loans[loanId]; is executed
        // So the same data saved from the loan will be used --> 300 WBTC
        uint256 loanId = loanIds[i];
        Loan memory loan = loans[loanId];
        (uint256 lenderInterest, uint256 protocolInterest) = _calculateInterest(loan);

        bytes32 poolId = getPoolId(loan.lender, loan.loanToken, loan.collateralToken);

        _updatePoolBalance(poolId, pools[poolId].poolBalance + loan.debt + lenderInterest);
        pools[poolId].outstandingLoans -= loan.debt;

        // 3️⃣ MB is losing STKN, but he is part of the scam so doen't matter to him.
        IERC20(loan.loanToken).transferFrom(msg.sender, address(this), loan.debt + lenderInterest);

        // 4️⃣ And here send another 300 WBTC from this contracts balance.
        // Malicious contract can detect the 2nd run and don't apply reentrancy
        // a 3rd time just in case the tx becomes too gas expensive for block validators.

        // 1️⃣ ❗❗ Critical Point: Malicious token triggers reentrancy in the repay()
        IERC20(loan.collateralToken).transfer(loan.borrower, loan.collateral);

        emit Repaid(
            msg.sender, loan.lender, loanId, loan.debt, loan.collateral, loan.interestRate, loan.startTimestamp
        );
        delete loans[loanId];
    }
}
```

Now the pool has 400 WBTC left and if the valid user tries to repay its debt there is no way the pool can give him back the 700 WBTC.

> 🚧 **Note** ⚠️: More sophisticated scams can employ genuine, value-bearing tokens as a decoy. They might lure non-technical users through a PROXY contract. This proxy seemingly manages the users' real tokens by redirecting all lending activities to the legitimate ERC20 token contract. However, before this redirection occurs, the proxy might sometimes execute malicious operations. This tactic creates an illusion, leading users to believe that their genuine tokens are being managed correctly. In reality, they're being manipulated through the PROXY contract.

 </details>

<details> <summary> borrow() and setPool() 🟡 </summary>

### _**`borrow()`**_ and _**`setPool()`**_

The problem with these 2 functions is essentially the same. They can be used to fill up the contract's storage with fake values effectively making it more expensive to operate with.

In `setPool()` users can create as much pools as they want and in `borrow()` users can push as much loans as they want to the loans array. Here **_reentrancy_** only helps them with creating more fake objects in storage with less transactions.

> 📘 **Notice** ℹ️: I've covered this storage exploit more detaildly and proposed solutions in other of my findings.

> 🚧 **Note** ⚠️ There might likely be more possible exploits with the reentrancies of these 2 functions.

 </details>

<details> <summary> buyLoan() 🟡 </summary>

### _**`buyLoan()`**_

Emits duplicate events, extra fees are paid, pools' debt can be doubled... Notice, in this case, reentrancy can be avoided just by simply applying the [Checks-Effects-Interactions pattern](https://docs.soliditylang.org/en/v0.6.11/security-considerations.html#re-entrancy).

```
    function buyLoan(uint256 loanId, bytes32 poolId) public {
        // get the loan info
        Loan memory loan = loans[loanId];

        // 2️⃣ (start reading on the emoji below saying 1 👁️)
        // As loans[loanId].auctionStartTimestamp = type(uint256).max; gets
        // executed after the reentrancy, this check would pass again.

        //validate the loan
        if (loan.auctionStartTimestamp == type(uint256).max) {
            revert AuctionNotStarted();
        }

        // more code

        // 3️⃣ totalDebt of the pool would be added twice, incorrect,
        // most probably will lead to even more exploits.
        pools[poolId].outstandingLoans += totalDebt;

        // more code

        // 4️⃣ Same here, pool loans would be substracted twice...
        pools[oldPoolId].outstandingLoans -= loan.debt;

        // 1️⃣❗❗ Reentrancy can happen here
        // transfer the protocol fee to the governance
        IERC20(loan.loanToken).transfer(feeReceiver, protocolInterest);

        emit Repaid(/*event args*/);

        // 5️⃣ Notice, this all happens because the state's update of the loan
        // is carried out after external calls, if we move this code to be executed
        // before them, then loan.auctionStartTimestamp would be the max
        // value and the reentrancy would revert in the if statement shown above (2️⃣).

        // update the loan with the new info
        loans[loanId].lender = msg.sender;
        loans[loanId].interestRate = pools[poolId].interestRate;
        loans[loanId].startTimestamp = block.timestamp;
        loans[loanId].auctionStartTimestamp = type(uint256).max;
        loans[loanId].debt = totalDebt;

        emit Borrowed( /* arguments... */ );
        emit LoanBought(loanId);
    }
```

 </details>

<details> <summary> zapBuyLoan() 🟡 </summary>

### _**`zapBuyLoan()`**_

As `zapBuyLoan()` calls `buyLoan()` and `setPool()`, it can be used to execute the exploits mentioned in those functions.

 </details>

<details> <summary> refinance(), seizeLoan() and giveLoan() 🟡 </summary>

### _**`refinance()`**_ , _**`seizeLoan()`**_ and _**`giveLoan()`**_

Due to timing reasons, I cannot provide detailed examples for these other 3 functions, but they manage funds and suffer from reentrancy, given the explained errors along the report they are highly likely to be a source for scammed funds or directly, stolen funds.

 </details>

<details> <summary> addToPool() and removeFromPool() ⚪ </summary>

### _**`addToPool()`**_ and _**`removeFromPool()`**_

I couldn't find any exploit using these functions but reentrancy is posible on them.

 </details>

---

## Impact 📈

The impact is fatal:

- System malfunctions. 🔻
- Financial losses for users. 🔻
- Duplicate system logs. 🔻
- High severity repercussions. 🔻

---

## Tools Used 🛠️

- Manual audit.
- Slither syntax analyzer.
- Solidity visual developer function dependency graph.

---

## Recommendations 🎯

- Incorporate OpenZeppelin's [ReentrancyGuard](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard) to the functions mentioned. Even to those flagged with ⚪, given the system's complexity, I highly recommend taking this precaution.

- Where appropriate, use the Checks-Effects-Interactions pattern because it provides a less gas-intensive solution than `ReentrancyGuard`.

- Enhance code's tests to detect these vulnerabilities. For that you can implement ERC20 mock contracts that apply reentrancy. There are OpenZeppelin codes that can be used for that or guide you in their implementation:

  - [ReentrancyMock.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/mocks/ReentrancyMock.sol)

  - [ReentrancyAttack.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/mocks/ReentrancyAttack.sol)

> 🚧 **Note** ⚠️ After changes are made, have another audit.
