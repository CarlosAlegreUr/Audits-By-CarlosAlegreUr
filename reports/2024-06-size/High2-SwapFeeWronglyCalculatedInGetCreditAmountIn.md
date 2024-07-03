# High Risk Issue: `Size` 🕵️

---

# **Users can pay less fees for the same amount of cash, protocol losses expected income**

---

### **Explanation && Impact** 📌📈

On the ***PoC-1*** below you will see that for the same tenor and same APR:

|               | `OPTION 1` | `OPTION 2` |
| ------------- | ---------- | ---------- |
| cash borrowed | 99_500000  | 100_000000 |
| Swap Fees     | 0_500000   | 0_500000   |

- _OPTION 1_: Bob puts a borrow offer, then Alice uses `buyCreditMarket` action
- _OPTION 2_: Alice puts lending offer, then Bob uses `sellCreditMarket` action

The problem arises from `sellCreditMarket` where the fees are calculated on the `cashAmountOut`, which is expected to have deducted from it the fees. `cashAmountOut` is actually the result of applying the fees to some amount **X**. Yet it is treated as if it were the amount **X** inside the function `getCreditAmountIn()`. And `X > cashAmountOut`, thus the proportional fees calculated on top of **X** will be bigger than the ones calcualted on `cashAmountOut`. So, as `cashAmountOut` is used in `getCreditAmountIn()`, that is why you are charging less fees for bigger amounts of cash.

In _OPTION-2_, you are calculating [here](https://github.com/code-423n4/2024-06-size/blob/main/src/libraries/AccountingLibrary.sol#L249) the fees on an amount which has already fees deducted from it, thus calculating them in a smaller amount that the actual amount they should be calculated from. Which is X => `X = cashAmountOut / (1-swapFee)`.

> ℹ️ **Note** 📘 You can deduce `X = cashAmountOut / (1-swapFee)` from `X - swapFee*X = cashAmountOut`. This equation is basically answering the question, if `cashAmountOut` came from something **X** being deducted swapFee, what was that something **X**?

> ℹ️ **Note** 📘 We know that in `sellCreditMarket`, `cashAmountOut` is meant to have fees deducted as this amount is directly transfered to the borrower at the end of the function. [See here](https://github.com/code-423n4/2024-06-size/blob/main/src/libraries/actions/SellCreditMarket.sol#L201). And the borrower pays `swapFee` due to the _cash-for-credit_ operation he is executing thus the amount he ends up receiving should be an amount with already deducted `swapFee` from.

The impact of this issue is pretty bad for the protocol as it receives less fees than the expected ones. Users can get more credit for the same amount of fees payed. This is a loss for the protocol as they are receiving less fees than they should.

Same happens when exiting loans via `sellCreditMarket` (***PoC-2***) when there is a partial exit. The lender of the position being exited can choose how much `swapFees` to be charged:

|               | `OPTION 1` | `OPTION 2` |
| ------------- | ---------- | ---------- |
| credit exited | 50_000000  | 50_000000  |
| Total Fees    | 5_681819   | 5_655683   |

When partially exiting the impact is also less fees charged and those fees not-charged are extra earnings for the user who makes a _credit-for-cash_ operation (in ***PoC-2*** that user is Candy, at _OPTION-2_). This is a loss for the protocol and a user receiving more money that she should.

---

### **Proof Of Concept (PoC)** 👨‍💻💻

Paste the tests below it in some test file `(./test/local/actions/*.t.sol)` on the system, import foundry _console.log_ in that file: `import "forge-std/console.sol";`. And run:

```bash
# For the first PoC
forge test --match-test "test_feecheckReservedCash" -vv
```

Before running the second PoC with:

```bash
forge test --match-test "test_feecheckExitW" -vv
```

You must paste this function inside `BaseTest.sol` contract, [here](https://github.com/code-423n4/2024-06-size/blob/main/test/BaseTest.sol#L185):


<details> <summary> See function 👁️ </summary>

```solidity
function _sellCreditMarketMINE(
        address borrower,
        address lender,
        uint256 creditPositionId,
        uint256 amount,
        uint256 tenor,
        bool exactAmountIn
    ) internal returns (uint256) {
        return _sellCreditMarket(borrower, lender, creditPositionId, amount, tenor, exactAmountIn);
    }
```

</details>

<details> <summary> See PoCs 👁️ </summary>

> ℹ️ **Note** 📘 If the test file you paste these test in requires more imports please read linter warnings about missing imports and add them.

<details> <summary> See PoC-1: Cheaper credit creating position 👁️ </summary>

```solidity
    function test_feecheckReservedCash() public {
        _deposit(alice, weth, 500e18);
        _deposit(alice, usdc, 500e6);
        _deposit(bob, weth, 500e18);
        _deposit(bob, usdc, 500e6);

        uint256[] memory tenors = new uint256[](2);
        tenors[0] = 365 days;
        tenors[1] = 365 * 2 days;
        int256[] memory aprs = new int256[](2);
        aprs[0] = 1.01e18;
        aprs[1] = 1.02e18;
        uint256[] memory marketRateMultipliers = new uint256[](2);

         Vars memory v = _state();
        uint256 bobInitalBalance = v.bob.borrowATokenBalance;

        // Option 1: Bob puts a borrow offer, then Alice calls buyCreditMarket
        _sellCreditLimit(bob, YieldCurve({tenors: tenors, aprs: aprs, marketRateMultipliers: marketRateMultipliers}));
        uint256 amount = 100e6;
        uint256 tenor = 365 days;
        uint256 debtPositionId = _buyCreditMarket(alice, bob, amount, tenor, true);
        uint256 credit = size.getDebtPosition(debtPositionId).futureValue;

        v = _state();
        console.log("----------------------------");
        console.log("OPTION 1: Bob puts a borrow offer, then Alice calls buyCreditMarket");
        console.log("borrower receives");
        console.log(v.bob.borrowATokenBalance - bobInitalBalance);
        console.log("lenders credit");
        console.log(credit);
        console.log("fees payed");
        console.log(v.feeRecipient.borrowATokenBalance);
    }

    function test_feecheckReservedCash2() public {
        _deposit(alice, weth, 500e18);
        _deposit(alice, usdc, 500e6);
        _deposit(bob, weth, 500e18);
        _deposit(bob, usdc, 500e6);

        uint256[2] memory tenors;
        tenors[0] = 365 days;
        tenors[1] = 365 * 2 days;
        int256[2] memory aprs;
        aprs[0] = 1.01e18;
        aprs[1] = 1.02e18;

        Vars memory v = _state();
        uint256 bobInitalBalance = v.bob.borrowATokenBalance;

        // Option 2: Alice puts lending offer, then Bob calls sellCreditMarket
        uint256 maxDueDate = 365 * 3 days;
        _buyCreditLimit(alice, maxDueDate, aprs, tenors);
        uint256 amount = 100e6; // 100 cash
        uint256 tenor = 365 days;
        uint256 debtPositionId = _sellCreditMarket(bob, alice, amount, tenor, false);
        uint256 credit = size.getDebtPosition(debtPositionId).futureValue;

        v = _state();
        console.log("----------------------------");
        console.log("OPTION 2: Alice puts lending offer, then Bob calls sellCreditMarket");
        console.log("RESULTS:");
        console.log("borrower receives");
        console.log(v.bob.borrowATokenBalance - bobInitalBalance);
        console.log("lenders credit");
        console.log(credit);
        console.log("fees payed");
        console.log(v.feeRecipient.borrowATokenBalance);

        console.log("+++++++++++++++++++++++++");
        console.log("FINAL RESUTLS OF GOAL: IN OPTION 1 BORROWER RECEIVES < 100$ YET PAYS SAME FEES");
        console.log("+++++++++++++++++++++++++");
    } 
```

</details>

<details> <summary> See PoC-2: Cheaper credit exiting position 👁️ </summary>

```solidity
        function test_feecheckExitWFragCredit2Active() public {
        console.log("------OPTION 1------");
        console.log("ALICE lends BOB: 100 credit");
        console.log("Alice will exit half of Alice's position: 50 credit");
        console.log("There will be swapFee and fragmentationFee. The exit is to a loan with the very same conditions.");
        console.log("swapFee=AliceShouldPay, fragmentationFee=AliceShouldPay");
        console.log(
            "NOTE AT THE END HOW THE TOTAL AMOUNT OF FEES PAYED IS BIGGER HERE, BUT THE FINAL INCREAE IN CREDIT AFTER CLAIM IS THE SAME"
        );
        console.log("--------------------");

        _deposit(alice, weth, 500e18);
        _deposit(alice, usdc, 500e6);
        _deposit(candy, weth, 500e18);
        _deposit(candy, usdc, 500e6);
        _deposit(bob, weth, 500e18);
        _deposit(bob, usdc, 500e6);

        uint256[] memory tenors = new uint256[](2);
        tenors[0] = 1 days;
        tenors[1] = 365 days;
        int256[] memory aprs = new int256[](2);
        aprs[0] = 0.01e18;
        aprs[1] = 0.1e18;
        uint256[] memory marketRateMultipliers = new uint256[](2);

        Vars memory v = _state();
        uint256 aliceBalancePrev = v.alice.borrowATokenBalance;
        uint256 candyBalancePrev = v.candy.borrowATokenBalance;

        // Bob puts a borrow offer, then Alice calls buyCreditMarket
        _sellCreditLimit(bob, YieldCurve({tenors: tenors, aprs: aprs, marketRateMultipliers: marketRateMultipliers}));
        uint256 amount = 100e6; // 100 credit
        uint256 tenor = tenors[1];
        uint256 debtPositionId = _buyCreditMarket(alice, bob, amount, tenor, false);
        (, uint256 creditCount) = size.getPositionsCount();
        uint256 aliceCreditPositionId = CREDIT_POSITION_ID_START + creditCount - 1;

        v = _state();
        console.log("----------------------------");
        console.log("RESULTS AFTER-LOAN but PRE-EXIT:");
        console.log("Alice balance:");
        console.log(v.alice.borrowATokenBalance);
        console.log("Candy balance:");
        console.log(v.candy.borrowATokenBalance);
        console.log("FeesRceiver balance:");
        console.log(v.feeRecipient.borrowATokenBalance);
        console.log("----------------------------");

        // To exit to Candy, Candy must have a lending offer. Notice tenors2 and aprs2 are exactly the same as before
        uint256 maxDueDate = 365 * 3 days;
        uint256[2] memory Ctenors;
        Ctenors[0] = tenors[0];
        Ctenors[1] = tenors[1];
        int256[2] memory Caprs;
        Caprs[0] = aprs[0];
        Caprs[1] = aprs[1];
        _buyCreditLimit(candy, maxDueDate, Caprs, Ctenors);
        _sellCreditMarketMINE(alice, candy, aliceCreditPositionId, amount / 2, type(uint256).max, true); // Alice exits half of it 50 credit
        (, creditCount) = size.getPositionsCount();
        uint256 candyCreditPositionId = CREDIT_POSITION_ID_START + creditCount - 1;

        v = _state();
        console.log("----------------------------");
        console.log("RESULTS AFTER-EXIT:");
        console.log("Alice balance:");
        console.log(v.alice.borrowATokenBalance);
        console.log("Candy balance:");
        console.log(v.candy.borrowATokenBalance);
        console.log("FeesRceiver balance:");
        console.log(v.feeRecipient.borrowATokenBalance);
        console.log("----------------------------");

        console.log("Now Bob is gonna repay and after, Alice and Candy claim their credit.");
        _repay(bob, debtPositionId);
        _claim(candy, candyCreditPositionId);
        _claim(alice, aliceCreditPositionId);
        v = _state();
        console.log("Alice:");
        console.log("Intial balance: ", aliceBalancePrev);
        console.log("Final balance : ", v.alice.borrowATokenBalance);

        console.log("Candy:");
        console.log("Intial balance: ", candyBalancePrev);
        console.log("Final balance : ", v.candy.borrowATokenBalance);
    }

    function test_feecheckExitWFragCash2Active() public {
        console.log("------OPTION 2------");
        console.log("ALICE lends BOB: 100 credit");
        console.log("Alice will exit half of Alice's position: 50 credit");
        console.log("There will be swapFee and fragmentationFee. The exit is to a loan with the very same conditions.");
        console.log("swapFee=AliceShouldPay, fragmentationFee=AliceShouldPay");
        console.log(
            "NOTE AT THE END HOW THE TOTAL AMOUNT OF FEES PAYED IS SMALLER HERE, BUT THE FINAL INCREAE IN CREDIT AFTER CLAIM IS THE SAME"
        );

        console.log("--------------------");

        _deposit(alice, weth, 500e18);
        _deposit(alice, usdc, 500e6);
        _deposit(candy, weth, 500e18);
        _deposit(candy, usdc, 500e6);
        _deposit(bob, weth, 500e18);
        _deposit(bob, usdc, 500e6);

        uint256[] memory tenors = new uint256[](2);
        tenors[0] = 1 days;
        tenors[1] = 365 days;
        int256[] memory aprs = new int256[](2);
        aprs[0] = 0.01e18;
        aprs[1] = 0.1e18;
        uint256[] memory marketRateMultipliers = new uint256[](2);

        Vars memory v = _state();
        uint256 aliceBalancePrev = v.alice.borrowATokenBalance;
        uint256 candyBalancePrev = v.candy.borrowATokenBalance;

        // Bob puts a borrow offer, then Alice calls buyCreditMarket
        _sellCreditLimit(bob, YieldCurve({tenors: tenors, aprs: aprs, marketRateMultipliers: marketRateMultipliers}));
        uint256 amount = 100e6; // 100 credit
        uint256 tenor = tenors[1];
        uint256 debtPositionId = _buyCreditMarket(alice, bob, amount, tenor, false);
        (, uint256 creditCount) = size.getPositionsCount();
        uint256 aliceCreditPositionId = CREDIT_POSITION_ID_START + creditCount - 1;

        v = _state();
        console.log("----------------------------");
        console.log("RESULTS AFTER-LOAN but PRE-EXIT:");
        console.log("Alice balance:");
        console.log(v.alice.borrowATokenBalance);
        console.log("Candy balance:");
        console.log(v.candy.borrowATokenBalance);
        console.log("FeesRceiver balance:");
        console.log(v.feeRecipient.borrowATokenBalance);
        console.log("----------------------------");

        // To exit to Candy, Candy must have a lending offer. Notice tenors2 and aprs2 are exactly the same as before
        uint256 maxDueDate = 365 * 3 days;
        uint256[2] memory Ctenors;
        Ctenors[0] = tenors[0];
        Ctenors[1] = tenors[1];
        int256[2] memory Caprs;
        Caprs[0] = aprs[0];
        Caprs[1] = aprs[1];
        _buyCreditLimit(candy, maxDueDate, Caprs, Ctenors);
        // Amount taken from previous valid test: 40.227272$, which corresponds with selling 50 credit
        // Alice wishes to receive 40.227272
        _sellCreditMarketMINE(alice, candy, aliceCreditPositionId, 40227272, type(uint256).max, false); // Alice exits half of it 50 credit, signaled in cash
        (, creditCount) = size.getPositionsCount();
        uint256 candyCreditPositionId = CREDIT_POSITION_ID_START + creditCount - 1;

        v = _state();
        console.log("----------------------------");
        console.log("RESULTS AFTER-EXIT:");
        console.log("Alice balance:");
        console.log(v.alice.borrowATokenBalance);
        console.log("Candy balance:");
        console.log(v.candy.borrowATokenBalance);
        console.log("FeesRceiver balance:");
        console.log(v.feeRecipient.borrowATokenBalance);
        console.log("----------------------------");

        console.log("Now Bob is gonna repay and after, Alice and Candy claim their credit.");
        _repay(bob, debtPositionId);
        _claim(candy, candyCreditPositionId);
        _claim(alice, aliceCreditPositionId);
        v = _state();
        console.log("Alice:");
        console.log("Intial balance: ", aliceBalancePrev);
        console.log("Final balance : ", v.alice.borrowATokenBalance);

        console.log("Candy:");
        console.log("Intial balance: ", candyBalancePrev);
        console.log("Final balance : ", v.candy.borrowATokenBalance);

        console.log("+++++++++++++++++++++++++");
        console.log("FINAL RESUTLS OF GOAL: ");
        console.log("+++++++++++++++++++++++++");
    }
```

</details>
</details>

---

### **Recommended Actions** 🛠️

> ⚠️ **Note** 🚧 After applying these reocomendations the PoCs provided work as expected, yet 7 test from the original `forge test` will fail. This is not due to the recommendation being wrong or the issue not true, it is due to the very same tests being wrong. The way is wrong is that their `asserts` check in the users' balances agains an increase/decrease of an amount calculated using a homemade calculated `swapFee`. Which is calculated from the same amount they use as input, and the core issue is that the very same amount you use as input is expected to already have fees deducted from it, not to calculate fees from it and then deduct them. See for example [this assertion](https://github.com/code-423n4/2024-06-size/blob/main/test/local/actions/SellCreditMarket.t.sol#L404), which uses to `50e6`, the same amount passed as `params.amount` a few lines above [here](https://github.com/code-423n4/2024-06-size/blob/main/test/local/actions/SellCreditMarket.t.sol#L400C57-L400C60). But this 50 indicates the exact amount of cash that Bob (in this test) wishes to receive, which is the amount Bob has to receive after all fees have been deducted. The other 6 tests that fail actually fail for the same reasons and I've listed them at the end of the issue report.

> ⚠️ **Note** 🚧 Even though I checked that all tests revert for this right reason, I recommend the dev team to double-check that this way of fixing the issue makes sense. 


At `getCreditAmountIn()` you should calculate the `swapFees` on the amount that did not have fees deducted from it yet. Thus you should calculate the fees on some `X = cashAmountOut / (1-swapFeePercent)`. Something like `fees = Math.mulDivUp(X, swapFeePercent, PERCENT);`.

And, in the part of the function that deals with fragmentation, fragmentation fee should also be taken into acount when figuring out the **X**.

I've deduced the fixes from the code docs and logic (using the notation they use in their docs, [see here](https://docs.size.credit/technical-docs/contracts/3.3-market-orders#id-3.3.1-sell-credit-market-order-borrowing-market-order)):

Here is the code I used to fix the issue:

```diff
    function getCreditAmountIn(
        State storage state,
        uint256 cashAmountOut,
        uint256 maxCashAmountOut,
        uint256 maxCredit,
        uint256 ratePerTenor,
        uint256 tenor
    ) internal view returns (uint256 creditAmountIn, uint256 fees) {
        uint256 swapFeePercent = getSwapFeePercent(state, tenor);

        uint256 maxCashAmountOutFragmentation = 0;

        if (maxCashAmountOut >= state.feeConfig.fragmentationFee) {
            maxCashAmountOutFragmentation = maxCashAmountOut - state.feeConfig.fragmentationFee;
        }

        // slither-disable-next-line incorrect-equality
        if (cashAmountOut == maxCashAmountOut) {
            // no credit fractionalization

+           // V before fees = maxCredit / (1+ratePerTenor)
+           uint256 discountedCashValue = Math.mulDivUp(maxCredit, PERCENT, PERCENT + ratePerTenor);

            creditAmountIn = maxCredit;
-           fees = Math.mulDivUp(cashAmountOut, swapFeePercent, PERCENT); // ORIGINAL
+           fees = Math.mulDivUp(discountedCashValue, swapFeePercent, PERCENT);
        } else if (cashAmountOut < maxCashAmountOutFragmentation) {
            // credit fractionalization

+           // Notice here we must take into account the frag fee, but creditAmountIn is already correctly calculated with it
            creditAmountIn = Math.mulDivUp(
                cashAmountOut + state.feeConfig.fragmentationFee, PERCENT + ratePerTenor, PERCENT - swapFeePercent
            );

+           uint256 discountedCashValue = Math.mulDivUp(creditAmountIn, PERCENT, PERCENT + ratePerTenor);
+           uint256 correctSwapFee = Math.mulDivUp(discountedCashValue, swapFeePercent, PERCENT);
-           fees = Math.mulDivUp(cashAmountOut, swapFeePercent, PERCENT) + state.feeConfig.fragmentationFee; // ORIGINAL
+            fees = correctSwapFee + state.feeConfig.fragmentationFee;
        } else {
            // for maxCashAmountOutFragmentation < amountOut < maxCashAmountOut we are in an inconsistent situation
            //   where charging the swap fee would require to sell a credit that exceeds the max possible credit

            revert Errors.NOT_ENOUGH_CASH(maxCashAmountOutFragmentation, cashAmountOut);
        }
    }
```

All 7 test that will revert but it's okay:

- testFuzz_SellCreditMarket_sellCreditMarket_exactAmountOut_properties
- testFuzz_SellCreditMarket_sellCreditMarket_used_to_borrow
- test_SellCreditMarket_sellCreditMarket_used_to_borrow
- test_Multicall_multicall_bypasses_cap_if_it_is_to_reduce_debt
- test_Liquidate_liquidate_can_be_called_unprofitably_and_liquidator_is_senior_creditor
- test_Compensate_compensate_used_to_borrower_exit_before_tenor_1
- test_SellCreditMarket_sellCreditMarket_exactAmountOut_numeric_example
