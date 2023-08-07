# **Summary** 📌

This report aims to enhance adherence to the programming paradigm:

- **Don't Repeat Yourself (DRY)** 📣

---

# **Vulnerability Details** 🔍

## In `Lender.sol`

<details>
    <summary> 🆕 New Modifier 🆕 </summary>

1. **New Modifier**:
   Apply this modifier to these functions: `addToPool()`, `removeFromPool()`, `updateMaxLoanRatio()`, `updateInterestRate()`.

   ```
   modifier callerIsLender(address lender) {
       require(lender == msg.sender, "Unauthorized");
       _;
   }
   ```

</details>

<details>
    <summary> 🆕 New Private Helper Functions 🆕 </summary>

2. **New Private Helper Functions**:

   Define the following functions in `Lender.sol` and use them in the following lines:

   At lines `369` and `613` implement a new function called `checkSameTokens()`:

   ```
   function checkSameTokens(Pool memory pool, Loan memory loan) private pure {
       if (pool.loanToken != loan.loanToken) revert TokenMismatch();
       if (pool.collateralToken != loan.collateralToken) {
           revert TokenMismatch();
       }
   }
   ```

   At lines `242` and `616` implement a new function called `checkDebtSizeInPool()`:

   ```
   function checkDebtSizeInPool(Pool memory pool, uint256 debt) private pure {
       if (debt < pool.minLoanSize) revert LoanTooSmall();
       if (debt > pool.poolBalance) revert LoanTooLarge();
   }
   ```

   At lines `246`, `384` and `619` implement a new function called `calculateAndCheckValidLoanRatio()`:

   ```
   function calculateAndCheckValidLoanRatio(Pool memory pool, uint256 debt, uint256 collateral) private pure {
       uint256 loanRatio = (debt * 10 ** 18) / collateral;
       if (loanRatio > pool.maxLoanRatio) revert RatioTooHigh();
   }
   ```

</details>

<details>
    <summary> Little change in refinance() 🔨 </summary>

3. **Adjustment in `refinance()`**:
   Line `595`.

   Replace the keccak256 encoding to get the pool ID for the already existing `getPoolId()` function.

   ```
   bytes32 oldPoolId = getPoolId(loans[loanId].lender, loans[loanId].loanToken, loans[loanId].collateralToken);
   ```

</details>

---

# **Impact** 📈

These changes not only enhance the use of the DRY paradigm but also bring an overall improvement of **0.01%** in the client's test gas consumption.

<details>
    <summary> Gas consumption details ⛽ 🚗 </summary>

<blockquote>

<details>
    <summary> Test by test breakdown 🧑‍🔬 </summary>

> 🚧 **Note** ⚠️: All tests but `LenderTest:test_createPool` have been included due to gas consumption changes being detected on them.

| _Test Name_                            | _Optimized Gas_ | _Original Gas_ | _Gas Saved_ |
| -------------------------------------- | --------------- | -------------- | ----------- |
| LenderTest:test_startAuction           | 626,592         | 626,519        | -73         |
| LenderTest:testFuzz_buyLoan            | 822,547         | 822,978        | 431         |
| LenderTest:testFuzz_repay              | 509,976         | 509,918        | -58         |
| LenderTest:testFail_borrowTooLarge     | 246,285         | 246,264        | -21         |
| LenderTest:test_borrow                 | 616,362         | 616,289        | -73         |
| LenderTest:testFuzz_seize              | 603,516         | 604,712        | 1,196       |
| LenderTest:testFuzz_borrow             | 247,726         | 247,705        | -21         |
| LenderTest:test_giveLoan               | 854,193         | 853,996        | -197        |
| LenderTest:test_seize                  | 546,648         | 546,589        | -59         |
| LenderTest:testFuzz_refinance          | 808,686         | 808,483        | -203        |
| LenderTest:testFail_repayNoTokens      | 617,851         | 617,778        | -73         |
| LenderTest:testFail_buyLoanTooLate     | 834,746         | 834,673        | -73         |
| LenderTest:testFail_buyLoanRateTooHigh | 835,244         | 835,171        | -73         |
| LenderTest:test_repay                  | 523,992         | 523,934        | -58         |
| LenderTest:test_buyLoan                | 857,779         | 857,706        | -73         |
| LenderTest:testFuzz_createPool         | 102,898         | 104,646        | 1,748       |
| LenderTest:testFail_borrowTooSmall     | 246,349         | 246,328        | -21         |
| LenderTest:test_refinance              | 850,486         | 850,199        | -287        |
| LenderTest:testFail_startAuction       | 621,961         | 621,888        | -73         |
| LenderTest:testFail_seizeTooEarly      | 632,373         | 632,300        | -73         |
| LenderTest:test_interest               | 622,282         | 622,209        | -73         |
| **TOTAL**                              | 12,382,620      | 12,389,631     | 1,793       |

Total saved percentage => **0.01%**.

> 📘 **Notice** ℹ️: The percentage has been calculated with these numbers from the TOTAL:
>
> ( 1,793 / 12,389,631 ) \* 100
>
> They mean:
>
> (totalGasSaved / originalGasCost) \* 100

</details>

<details>
        <summary> Forge Snapshots Used 📸 </summary>

_**`Original`**_

```
LenderTest:testFail_borrowTooLarge() (gas: 246264)
LenderTest:testFail_borrowTooSmall() (gas: 246328)
LenderTest:testFail_buyLoanRateTooHigh() (gas: 835171)
LenderTest:testFail_buyLoanTooLate() (gas: 834673)
LenderTest:testFail_repayNoTokens() (gas: 617778)
LenderTest:testFail_seizeTooEarly() (gas: 632300)
LenderTest:testFail_startAuction() (gas: 621888)
LenderTest:testFuzz_borrow(uint256,uint256) (runs: 256, μ: 247705, ~: 247702)
LenderTest:testFuzz_buyLoan(uint256) (runs: 256, μ: 822978, ~: 833057)
LenderTest:testFuzz_createPool(uint256,uint256) (runs: 256, μ: 104646, ~: 22504)
LenderTest:testFuzz_refinance(uint256,uint256) (runs: 256, μ: 808483, ~: 808483)
LenderTest:testFuzz_repay(uint256) (runs: 256, μ: 509918, ~: 509917)
LenderTest:testFuzz_seize(uint256) (runs: 256, μ: 604712, ~: 605664)
LenderTest:test_borrow() (gas: 616289)
LenderTest:test_buyLoan() (gas: 857706)
LenderTest:test_createPool() (gas: 240654)
LenderTest:test_giveLoan() (gas: 853996)
LenderTest:test_interest() (gas: 622209)
LenderTest:test_refinance() (gas: 850199)
LenderTest:test_repay() (gas: 523934)
LenderTest:test_seize() (gas: 546589)
LenderTest:test_startAuction() (gas: 626519)
```

_**`Optimized`**_

```
LenderTest:test_startAuction | 626592 | 626519 | -73
LenderTest:testFuzz_buyLoan | 822547 | 822978 | 431
LenderTest:testFuzz_repay | 509976 | 509918 | -58
LenderTest:testFail_borrowTooLarge | 246285 | 246264 | -21
LenderTest:test_borrow | 616362 | 616289 | -73
LenderTest:testFuzz_seize | 603516 | 604712 | 1196
LenderTest:testFuzz_borrow | 247726 | 247705 | -21
LenderTest:test_giveLoan | 854193 | 853996 | -197
LenderTest:test_seize | 546648 | 546589 | -59
LenderTest:test_createPool | 240654 | 240654 | 0
LenderTest:testFuzz_refinance | 808686 | 808483 | -203
LenderTest:testFail_repayNoTokens | 617851 | 617778 | -73
LenderTest:testFail_buyLoanTooLate | 834746 | 834673 | -73
LenderTest:testFail_buyLoanRateTooHigh | 835244 | 835171 | -73
LenderTest:test_repay | 523992 | 523934 | -58
LenderTest:test_buyLoan | 857779 | 857706 | -73
LenderTest:testFuzz_createPool | 102898 | 104646 | 1748
LenderTest:testFail_borrowTooSmall | 246349 | 246328 | -21
LenderTest:test_refinance | 850486 | 850199 | -287
LenderTest:testFail_startAuction | 621961 | 621888 | -73
LenderTest:testFail_seizeTooEarly | 632373 | 632300 | -73
LenderTest:test_interest | 622282 | 622209 | -73
```

</details>

> 🚧 **Note** ⚠️: The figures presented represent total gas consumption. Not every individual modification necessarily contributes to gas savings. Some changes might increase gas costs, while others decrease them. However, the net effect can still result in overall reduced gas consumption. To fully understand the impact of each change, a more detailed, granular analysis is essential for each modification.

</blockquote>

</details>

---

# **Tools Used** 🛠️

- Manual audit.
- Solidity Visual Developer VSPlugin: Function dependencies graph.
- Forge Snapshot
- Bash scripts tailored to analyze forge snapshots.

> 📘 **Notice** ℹ️: I've personally created the bash scripts. Here is a link to the github repo [Forge-Snapshots-Analyzer](https://github.com/CarlosAlegreUr/Forge-Snapshots-Analyzer).

---

# **Recommendations** 🎯

Due to the improved readability, maintainability, and gas consumption, implementing the suggested changes is recommended.
