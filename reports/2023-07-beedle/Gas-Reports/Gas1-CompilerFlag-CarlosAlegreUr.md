# Summary 📌

Following recent updates, client-tests have passed, achieving gas savings of **0.18%**. This optimization review focused solely on the `Lender.sol` contract, the only one with associated tests.

---

# Vulnerability Details 🔍

### Use `via_ir = true` for an Enhanced Compiler Process 🛠️

The `via_ir` flag in the `foundry.toml` file triggers a more optimized compiler. This change resulted in a **0.18%** improvement over the unoptimized contract.

**`foundry.toml` Configuration**:

```
(options...)
remappings = [...]
via_ir = true  <-- Set flag for optimization
```

> 🚧 **Note** ⚠️: Enabling `via_ir` prolongs the compilation process. Compilation-related tasks are expected to take quite a bit more of time when this flag is active.

> 📘 **Information** ℹ️: The `via_ir` flag instructs Foundry to utilize Intermediate Representation (IR). IR is a low-level language bridging high-level source code and machine code. This intermediate step allows for additional optimization techniques. Thats what the `_ir` means in `via_ir`, Intermediate Representation.

---

# Impact 📈

Metrics derived from gas consumption of the provided tests:

| _Optimization Method_ | _Optimized Gas_ | _Original Gas_ | _Gas Saved_ | _% Saved_ |
| --------------------- | --------------- | -------------- | ----------- | --------- |
| **Use of via_ir**     | 12,870,939      | 12,847,388     | 23,551      | **0.18%** |

<details> 
<summary> Test by test breakdown 🧑‍🔬 </summary>

> 🚧 **Notice** ⚠️: The optimization affects all the contract, so I've analyzed all the tests.

> 🚧 **Notice** ⚠️: Negative numbers in the `Gas Saved` column means the "optimization" increased the consumption of gas in that test. But the overall outcome in this case is positive so it can be considered an optimization.

| Test Name                              | Optimized Gas | Original Gas | Gas Saved |
| -------------------------------------- | ------------- | ------------ | --------- |
| LenderTest:test_startAuction           | 626,458       | 626,519      | 61        |
| LenderTest:testFuzz_buyLoan            | 820,537       | 822,978      | 2,441     |
| LenderTest:testFuzz_repay              | 508,429       | 509,918      | 1,489     |
| LenderTest:testFail_borrowTooLarge     | 245,932       | 246,264      | 332       |
| LenderTest:test_borrow                 | 615,740       | 616,289      | 549       |
| LenderTest:testFuzz_seize              | 604,236       | 604,712      | 476       |
| LenderTest:testFuzz_borrow             | 247,162       | 247,705      | 543       |
| LenderTest:test_giveLoan               | 851,476       | 853,996      | 2,520     |
| LenderTest:test_seize                  | 546,439       | 546,589      | 150       |
| LenderTest:test_createPool             | 238,918       | 240,654      | 1,736     |
| LenderTest:testFuzz_refinance          | 807,460       | 808,483      | 1,023     |
| LenderTest:testFail_repayNoTokens      | 617,720       | 617,778      | 58        |
| LenderTest:testFail_buyLoanTooLate     | 832,601       | 834,673      | 2,072     |
| LenderTest:testFail_buyLoanRateTooHigh | 832,822       | 835,171      | 2,349     |
| LenderTest:test_repay                  | 522,975       | 523,934      | 959       |
| LenderTest:test_buyLoan                | 854,506       | 857,706      | 3,200     |
| LenderTest:testFuzz_createPool         | 105,058       | 104,646      | -412      |
| LenderTest:testFail_borrowTooSmall     | 245,483       | 246,328      | 845       |
| LenderTest:test_refinance              | 848,406       | 850,199      | 1,793     |
| LenderTest:testFail_startAuction       | 621,394       | 621,888      | 494       |
| LenderTest:testFail_seizeTooEarly      | 631,900       | 632,300      | 400       |
| LenderTest:test_interest               | 621,736       | 622,209      | 473       |
| **TOTAL**                              | 12,847,388    | 12,870,939   | 23,551    |

Total saved percentage => **0.18%**.

> 📘 **Notice** ℹ️: The percentage has been calculated with these numbers from the TOTAL:
>
> ( 23,551 / 12,870,939 ) \* 100
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
LenderTest:testFail_borrowTooLarge() (gas: 245932)
LenderTest:testFail_borrowTooSmall() (gas: 245483)
LenderTest:testFail_buyLoanRateTooHigh() (gas: 832822)
LenderTest:testFail_buyLoanTooLate() (gas: 832601)
LenderTest:testFail_repayNoTokens() (gas: 617720)
LenderTest:testFail_seizeTooEarly() (gas: 631900)
LenderTest:testFail_startAuction() (gas: 621394)
LenderTest:testFuzz_borrow(uint256,uint256) (runs: 256, μ: 247162, ~: 247159)
LenderTest:testFuzz_buyLoan(uint256) (runs: 256, μ: 820537, ~: 830195)
LenderTest:testFuzz_createPool(uint256,uint256) (runs: 256, μ: 105058, ~: 22578)
LenderTest:testFuzz_refinance(uint256,uint256) (runs: 256, μ: 807460, ~: 807460)
LenderTest:testFuzz_repay(uint256) (runs: 256, μ: 508429, ~: 508428)
LenderTest:testFuzz_seize(uint256) (runs: 256, μ: 604236, ~: 605190)
LenderTest:test_borrow() (gas: 615740)
LenderTest:test_buyLoan() (gas: 854506)
LenderTest:test_createPool() (gas: 238918)
LenderTest:test_giveLoan() (gas: 851476)
LenderTest:test_interest() (gas: 621736)
LenderTest:test_refinance() (gas: 848406)
LenderTest:test_repay() (gas: 522975)
LenderTest:test_seize() (gas: 546439)
LenderTest:test_startAuction() (gas: 626458)
```

 </details>

---

# Tools Used 🛠️

- Manual audit
- Forge Snapshot
- Bash scripts tailored to analyze forge snapshots.

> 📘 **Notice** ℹ️: I've personally created the bash scripts. Here is a link to the github repo [Forge-Snapshots-Analyzer](https://github.com/CarlosAlegreUr/Forge-Snapshots-Analyzer).

---

# Recommendations 🎯

Implementing the proposed gas optimizations is recommended. This benefits both end-users and the protocol in terms of cost, while ensuring protocol security remains uncompromised as tests pass consistently.

> 🚧 **Note** ⚠️: If a significant vulnerability necessitating major code changes arises, this gas optimization technique remains relevant and applicable. Nonetheless some gas analysis as the one used here is always needed to make sure it's actually saving gas.
