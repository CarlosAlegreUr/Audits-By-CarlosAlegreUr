## Summary 📌

Following the recent changes, the given client-tests have successfully passed, resulting in gas savings of around **0.155%**.

## Vulnerability Details 🔍

The gas optimization was made on the `Lender.sol` contract.

> 📘 **Notice** ℹ️: I've provided the optimized versions of the **`Lender.sol` functions** at the very end of this report.

### Storage Writes Optimization ✏️

In the methods `refinance()` and `giveLoan()`, there are several storage write operations which can be optimized. Specifically, lines 686-696 in the `refinance()` method are a prime example:

```
loans[loanId].collateral = collateral;
loans[loanId].interestRate = pool.interestRate;
loans[loanId].startTimestamp = block.timestamp;
loans[loanId].auctionStartTimestamp = type(uint256).max;
loans[loanId].auctionLength = pool.auctionLength;
loans[loanId].lender = pool.lender;
```

It's possible to leverage the loan variable declared in memory within this function, update the changes there, and eventually copy this memory variable to storage. This approach results in fewer storage write operations leading to less gas usage.

Here's an optimized approach for the `refinance()` method:

```
// 🟢 Already declared and used loan memory object
Loan memory loan = loans[loanId];
//... More code ...
loan.collateral = collateral;
loan.interestRate = pool.interestRate;
loan.startTimestamp = block.timestamp;
loan.auctionStartTimestamp = type(uint256).max;
loan.auctionLength = pool.auctionLength;
loan.lender = pool.lender;
loans[loanId] = loan; // <------ 🟢 This results in only a single storage write.
```

The `giveLoan()` function shares a similar context, and its changes can be reviewed in the optimized code appended at the end of the report.

> 📘 **Notice** ℹ️: I experimented applying this change to Pool memory objects too. But this didn't translate to gas savings but instead an increased gas consumption. This might be because writing just a couple of pool traits directly to storage may be more gas efficient than apply them with memory and subsequently save them to storage. This observation is intended to guide the development team in case they are thinking let's apply the same with the Pool object.

## Impact 📈

All impact metrics are from the gas consumption by the clients' provided tests.

| Optimization Method      | _Original Gas_ | _Optimized Gas_ | _Gas Saved_ | _% Saved_ |
| ------------------------ | -------------- | --------------- | ----------- | --------- |
| **Single Storage Write** | 3,138,664      | 3,444,012       | 5,348       | 0.155%    |

<details> 
<summary> Test by test breakdown 🧑‍🔬 </summary>

> 🚧 **Note** ⚠️: Only the test that have been affected by the change have been analyzed. The other tests had no variation in its gas consumption.

> 🚧 **Note** ⚠️: Negative numbers in the `Gas Saved` column means the "optimization" increased the consumption of gas in that test. But the overall outcome in this case is positive so it can be considered an optimization.

| Test Name                      | Optimized Gas | Original Gas | Gas Saved |
| ------------------------------ | ------------- | ------------ | --------- |
| LenderTest:testFuzz_buyLoan    | 822,040       | 822,978      | 938       |
| LenderTest:testFuzz_refinance  | 808,485       | 808,483      | -2        |
| LenderTest:test_buyLoan        | 857,664       | 857,706      | 42        |
| LenderTest:testFuzz_createPool | 100,509       | 104,646      | 4137      |
| LenderTest:test_refinance      | 849,966       | 850,199      | 233       |
| **TOTAL**                      | 3,438,664     | 3,444,012    | 5,348     |

Total saved percentage => **0.155%**.

> 📘 **Notice** ℹ️: The percentage has been calculated with these numbers from the TOTAL:
>
> ( 5,348 / 3,444,012 ) \* 100
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

## Recommendations 🎯

This gas omptimization is easy to implement and it will save some gas, therefore reduce the overall cost of the project. Thats why it's implementation is advised.

> **Notice** ⚠️: If a significant vulnerability necessitating major code changes arises, this gas optimization technique remains relevant and applicable. Nonetheless some gas analysis as the one used here is always needed to make sure it's actually saving gas.

<details>
<summary> Optimized Functions 📊 </summary>

_**`Lender.sol` functions**_

Changes are marked by the 🟢 emoji.

```
    function buyLoan(uint256 loanId, bytes32 poolId) public {
        // get the loan info
        Loan memory loan = loans[loanId];
        // validate the loan
        if (loan.auctionStartTimestamp == type(uint256).max) {
            revert AuctionNotStarted();
        }
        if (block.timestamp > loan.auctionStartTimestamp + loan.auctionLength) {
            revert AuctionEnded();
        }
        // calculate the current interest rate
        uint256 timeElapsed = block.timestamp - loan.auctionStartTimestamp;
        uint256 currentAuctionRate = (MAX_INTEREST_RATE * timeElapsed) / loan.auctionLength;
        // validate the rate
        if (pools[poolId].interestRate > currentAuctionRate) revert RateTooHigh();
        // calculate the interest
        (uint256 lenderInterest, uint256 protocolInterest) = _calculateInterest(loan);

        // reject if the pool is not big enough
        uint256 totalDebt = loan.debt + lenderInterest + protocolInterest;
        if (pools[poolId].poolBalance < totalDebt) revert PoolTooSmall();

        // if they do have a big enough pool then transfer from their pool
        _updatePoolBalance(poolId, pools[poolId].poolBalance - totalDebt);
        pools[poolId].outstandingLoans += totalDebt;

        // now update the pool balance of the old lender
        bytes32 oldPoolId = getPoolId(loan.lender, loan.loanToken, loan.collateralToken);
        _updatePoolBalance(oldPoolId, pools[oldPoolId].poolBalance + loan.debt + lenderInterest);
        pools[oldPoolId].outstandingLoans -= loan.debt;

        // transfer the protocol fee to the governance
        IERC20(loan.loanToken).transfer(feeReceiver, protocolInterest);

        emit Repaid(
            loan.borrower,
            loan.lender,
            loanId,
            loan.debt + lenderInterest + protocolInterest,
            loan.collateral,
            loan.interestRate,
            loan.startTimestamp
        );

        // 🟢 HERE IS THE CHANGE
        // update the loan with the new info
        loan.lender = msg.sender;
        loan.interestRate = pools[poolId].interestRate;
        loan.startTimestamp = block.timestamp;
        loan.auctionStartTimestamp = type(uint256).max;
        loan.debt = totalDebt;

        loans[loanId] = loan;

        emit Borrowed(
            loan.borrower, msg.sender, loanId, loan.debt, loan.collateral, pools[poolId].interestRate, block.timestamp
        );
        emit LoanBought(loanId);
    }

    function refinance(Refinance[] calldata refinances) public {
        for (uint256 i = 0; i < refinances.length; i++) {
            uint256 loanId = refinances[i].loanId;
            bytes32 poolId = refinances[i].poolId;
            bytes32 oldPoolId =
                keccak256(abi.encode(loans[loanId].lender, loans[loanId].loanToken, loans[loanId].collateralToken));
            uint256 debt = refinances[i].debt;
            uint256 collateral = refinances[i].collateral;

            // get the loan info
            Loan memory loan = loans[loanId];
            // validate the loan
            if (msg.sender != loan.borrower) revert Unauthorized();

            // get the pool info
            Pool memory pool = pools[poolId];
            // validate the new loan
            if (pool.loanToken != loan.loanToken) revert TokenMismatch();
            if (pool.collateralToken != loan.collateralToken) {
                revert TokenMismatch();
            }
            if (pool.poolBalance < debt) revert LoanTooLarge();
            if (debt < pool.minLoanSize) revert LoanTooSmall();
            uint256 loanRatio = (debt * 10 ** 18) / collateral;
            if (loanRatio > pool.maxLoanRatio) revert RatioTooHigh();

            // calculate the interest
            (uint256 lenderInterest, uint256 protocolInterest) = _calculateInterest(loan);
            uint256 debtToPay = loan.debt + lenderInterest + protocolInterest;

            // update the old lenders pool
            _updatePoolBalance(oldPoolId, pools[oldPoolId].poolBalance + loan.debt + lenderInterest);
            pools[oldPoolId].outstandingLoans -= loan.debt;

            // now lets deduct our tokens from the new pool
            _updatePoolBalance(poolId, pools[poolId].poolBalance - debt);
            pools[poolId].outstandingLoans += debt;

            if (debtToPay > debt) {
                // we owe more in debt so we need the borrower to give us more loan tokens
                // transfer the loan tokens from the borrower to the contract
                IERC20(loan.loanToken).transferFrom(msg.sender, address(this), debtToPay - debt);
            } else if (debtToPay < debt) {
                // we have excess loan tokens so we give some back to the borrower
                // first we take our borrower fee
                uint256 fee = (borrowerFee * (debt - debtToPay)) / 10000;
                IERC20(loan.loanToken).transfer(feeReceiver, fee);
                // transfer the loan tokens from the contract to the borrower
                IERC20(loan.loanToken).transfer(msg.sender, debt - debtToPay - fee);
            }
            // transfer the protocol fee to governance
            IERC20(loan.loanToken).transfer(feeReceiver, protocolInterest);

            // 🟢 Udpate things on the memory loan, then just do a single write to storage
            // loan.debt is not used anymore in the follwing code so no problem changing it
            // update loan debt
            loan.debt = debt;

            // update loan collateral
            if (collateral > loan.collateral) {
                // transfer the collateral tokens from the borrower to the contract
                IERC20(loan.collateralToken).transferFrom(msg.sender, address(this), collateral - loan.collateral);
            } else if (collateral < loan.collateral) {
                // transfer the collateral tokens from the contract to the borrower
                IERC20(loan.collateralToken).transfer(msg.sender, loan.collateral - collateral);
            }


            emit Repaid(msg.sender, loan.lender, loanId, debt, collateral, loan.interestRate, loan.startTimestamp);

            // More changes here! 🟢
            loan.collateral = collateral;
            // update loan interest rate
            loan.interestRate = pool.interestRate;
            // update loan start timestamp
            loan.startTimestamp = block.timestamp;
            // update loan auction start timestamp
            loan.auctionStartTimestamp = type(uint256).max;
            // update loan auction length
            loan.auctionLength = pool.auctionLength;
            // update loan lender
            loan.lender = pool.lender;

            loans[loanId] = loan;
            // update pool balance
            pools[poolId].poolBalance -= debt;
            emit Borrowed(msg.sender, pool.lender, loanId, debt, collateral, pool.interestRate, block.timestamp);
            emit Refinanced(loanId);
        }
    }
```
