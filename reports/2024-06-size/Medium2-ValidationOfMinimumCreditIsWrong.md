# Medium Risk Issue: `Size` 🕵️

---

# **Creation of valid loans will revert**

---

### **Explanation && Impact** 📌📈

At `BuyCreditMarket.sol` && `SellCreditMarket.sol` inside their respective validate functions: `validateBuyCreditMarket()` && `validateSellCreditMarket()`. There is the [following if statement](https://github.com/code-423n4/2024-06-size/blob/main/src/libraries/actions/SellCreditMarket.sol#L93):

```solidity
      // validate amount
        if (params.amount < state.riskConfig.minimumCreditBorrowAToken) {
            revert Errors.CREDIT_LOWER_THAN_MINIMUM_CREDIT(params.amount, state.riskConfig.minimumCreditBorrowAToken);
        }
```

This statement compares `params.amount` (that can represent credit or cash depeding on the `exactAmountIn` flag) with `state.riskConfig.minimumCreditBorrowAToken` which is always a credit specified amount. This can cause reverts on valid loans creation or exits.

- **Numeric example:**
  
**At buyCreditMarket** lets say `minimumCreditBorrowAToken == 50` lender calls `buyCreditMarket` and specifies that the amount of cash you want to send to the borrower is 49$. A borrower accepts this with an interest rate of 3%, future credit of the loan is `49 * 1.03 = 50.47 $`. Which is greater than `minimumCreditBorrowAToken` but the loan is not created because of the comparison in the validate function reverting. Same at **sellCreditMarket**, a borrower specifies that the amount of cash he wants to pay after fees is 49$, which at an interest rate of 3% is `futureValue=50.47 $` which is greater than `minimumCreditBorrowAToken` but the loan is not created because of the comparison in the validate function. Same goes in both functions when `RESERVED_ID` is not passed, the comparison can cause reverts when exiting valid loans where the discounted cash value is < minimumCreditBorrowAToken.

The impacts are:

It is true that you can still create this loans by passing `exactAmountIn==FALSE` and specifying the credit amount that would give you the desired $ amount. But this is more complex dev experience plus prone to errors and also for protocols that might want to build on top of Size, this can be a problem as they expect all API/functions interactions with valid inputs to not revert. The system reverts in valid txs and validates its input incorrectly.

---

### **Proof Of Concept (PoC)** 👨‍💻💻

Paste the code below it in some test file `(./test/local/actions/*.t.sol)` on the system, import foundry _console.log_ in that file: `import "forge-std/console.sol";`. And run:

```bash
# To see how this affects position creations
forge test --match-test "testFail_minimChecks" -vvv

# To see how this affects positon exits
forge test --match-test "test_minFragCheckExitP" -vv
```

Also add _console.log_ to `BuyCreditMarket.sol` inside `validateBuyCreditMarket()` like so:

First line to add _console.log_: [L-90](https://github.com/code-423n4/2024-06-size/blob/main/src/libraries/actions/BuyCreditMarket.sol#L90)

Second line: [L-94](https://github.com/code-423n4/2024-06-size/blob/main/src/libraries/actions/BuyCreditMarket.sol#L94)

Add the same in the respective lines of `validateSellCreditMarket()` in `SellCreditMarket.sol`: [here](https://github.com/code-423n4/2024-06-size/blob/main/src/libraries/actions/SellCreditMarket.sol#L93).

```diff
+ // 🟢 Example on buyCreditMarket, do the same in sellCreditMarket 🟢 
function validateBuyCreditMarket(State storage state, BuyCreditMarketParams calldata params) external view {
    // function code...
        // validate amount
+       console.log("It reverts here!");
        if (params.amount < state.riskConfig.minimumCreditBorrowAToken) {
            revert Errors.CREDIT_LOWER_THAN_MINIMUM_CREDIT(params.amount, state.riskConfig.minimumCreditBorrowAToken);
        }
+       console.log("This will not appear!");
      // rest of function code...
}
```

For seeing the PoC on positions' exits you need an extra function in `BaseTest.sol` contract, [here](https://github.com/code-423n4/2024-06-size/blob/main/test/BaseTest.sol#L185):

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

<details> <summary> See code PoCs 👁️ </summary>

> ⚠️ **Note** 🚧 Notice that the `testFail_minimChecks` test is a Foundry `testFail_`, if it passes is because the transaction reverted. Which is the thing we want to proove. Also notice that from the 2 _console.log_ in `buyCreditMarket` only 1 should be printed.

> ℹ️ **Note** 📘 If the test file you paste this test in requires more imports please read linter warnings about missing imports and add them.

```solidity
    function testFail_minimChecks() public {
        _deposit(alice, weth, 500e18);
        _deposit(alice, usdc, 500e6);
        _deposit(bob, weth, 500e18);
        _deposit(bob, usdc, 500e6);

        uint256[] memory tenors = new uint256[](2);
        tenors[0] = 128 days;
        tenors[1] = 365 days;
        int256[] memory aprs = new int256[](2);
        aprs[0] = 0.01e18;
        aprs[1] = 0.03e18;
        uint256[] memory marketRateMultipliers = new uint256[](2);

        // Bob puts a borrow offer, then Alice calls buyCreditMarket
        _sellCreditLimit(bob, YieldCurve({tenors: tenors, aprs: aprs, marketRateMultipliers: marketRateMultipliers}));

        console.log("We are creating a loan of 4.9$ at a 3% interest rate = 4.9*1.03 = 5.047 credit.");
        console.log("This amount of credit is bigger than the minimumCreditBorrowAToken.");

        uint256 amount = size.riskConfig().minimumCreditBorrowAToken - 1e5; // we specify 4.9$ cash, which is 5.047 credit, should pass
        console.log("The minimumCreditBorrowAToken is: ", size.riskConfig().minimumCreditBorrowAToken);
        console.log("Amount of cah as input:           ", amount);
        console.log("Input plus interests:             ", Math.mulDivDown(amount, PERCENT + uint256(aprs[1]), PERCENT));
        uint256 tenor = tenors[1];
        _buyCreditMarket(alice, bob, amount, tenor, true); // true == cash specifed
    }

    function test_minFragCheckExitP() public {
        console.log("--------SCENARIO 1-----------");
        console.log("ALICE lends BOB: 100 credit");
        console.log("Alice will exit Alice's position to Candy.");
        console.log("There will be swapFee and fragmentationFee. The exit is to a loan with the very same conditions.");
        console.log("swapFee=AliceShouldPay, fragmentationFee=AliceShouldPay");
        console.log("--------------------");
        console.log("Conditions of the loan && exit loan are the same:");
        console.log("1 YEAR -> APR 0.1 -> ratePerTenor 0.1");
        console.log(
            "swapFee amount, as 0.5% a year and these loans are 1 YEAR, it will always be 0.005 * discountedCashValue"
        );
        console.log("fragmentationFee is set to 5$");
        console.log("Credit amount is 100");
        console.log("minimumCreditBorrowAToken: 5$");
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

        // Bob puts a borrow offer, then Alice calls buyCreditMarket
        _sellCreditLimit(bob, YieldCurve({tenors: tenors, aprs: aprs, marketRateMultipliers: marketRateMultipliers}));
        uint256 amount = 100e6; // 100 credit
        uint256 tenor = tenors[1];
        uint256 debtPositionId = _buyCreditMarket(alice, bob, amount, tenor, false);
        (, uint256 creditCount) = size.getPositionsCount();
        uint256 aliceCreditPositionId = CREDIT_POSITION_ID_START + creditCount - 1;

        // To exit to Candy, Candy must have a lending offer. Notice tenors2 and aprs2 are exactly the same as before
        uint256 maxDueDate = 365 * 3 days;
        uint256[2] memory Ctenors;
        Ctenors[0] = tenors[0];
        Ctenors[1] = tenors[1];
        int256[2] memory Caprs;
        Caprs[0] = aprs[0];
        Caprs[1] = aprs[1];
        _buyCreditLimit(candy, maxDueDate, Caprs, Ctenors);

        Vars memory v = _state();
        uint256 back = vm.snapshot();

        console.log(
            "Notice, Alice can't sell 5 credit to candy. Because the amount she receives will be negative as she is paying fragFee:"
        );
        console.log("V = (5/1.1)*(1-0.005)-5 = -0.477272");
        console.log("So what is the minimal amount that Alice can exit to Candy without negative balance?");
        console.log("(using 6 decimals of precision as credit has 6 decimals)");
        console.log("0 = (A/1.1)*(1-0.005)-5 -----> A = 5.527639");
        console.log("We now execute the exit with amount=5527639");
        _sellCreditMarketMINE(alice, candy, aliceCreditPositionId, 5527639, type(uint256).max, true); // Alice exits part of its credit
        (, creditCount) = size.getPositionsCount();
        uint256 candyCreditPositionId = CREDIT_POSITION_ID_START + creditCount - 1;
        console.log("And you can see it passes.");

        v = _state();
        uint256 ap = v.alice.borrowATokenBalance;
        uint256 cp = v.candy.borrowATokenBalance;
        console.log("Now Bob is gonna repay and after, Alice and Candy claim their credit.");
        _repay(bob, debtPositionId);
        _claim(candy, candyCreditPositionId);
        _claim(alice, aliceCreditPositionId);
        v = _state();
        console.log("Alice exited 5.527639 to Candy, so Alice after claim should have 100-5.527639=94.472361");
        console.log("Credit increase after claim for Alice: ", v.alice.borrowATokenBalance - ap);
        console.log("Credit increase after claim for Candy: ", v.candy.borrowATokenBalance - cp);
        console.log("ALL CORRECT AND FINE HERE! But, what if Alice makes this operation signaling amount in cash?");

        vm.revertTo(back);
        console.log("--------SCENARIO 2-----------");
        console.log("Same as scenario 1, but this time Alice will exit signaling the params.amount in cash");
        console.log("--------------------");

        console.log("Remember from before:");
        console.log("0 = (A/1.1)*(1-0.005)-5 -----> A = 5.527639");
        console.log("So Alice receiving 0 cash comes from a fragmentation of credit to a position of 5.527639");
        console.log("Which is clearly bigger than 5, remember 5 is minimumCreditBorrowAToken.");
        console.log("We now execute the exit with amount=0");
        console.log("For the reasons above this should execute, but it does not.");
        console.log("Executing...");
        vm.expectRevert();
        _sellCreditMarketMINE(alice, candy, aliceCreditPositionId, 0, type(uint256).max, false); // Alice exits part of its credit, signaled in cash
        console.log("The expectRevert() was successfull, thus it reverted.");
        console.log("You should see just the result of: It reverts here!. 2 lines above this one.");

        console.log("Now lets pass 1 as the cash amount, which should correspond to:");
        console.log("0.0000001 = (A/1.1)*(1-0.005)-5 ----> A=5.527640");
        vm.expectRevert();
        _sellCreditMarketMINE(alice, candy, aliceCreditPositionId, 1, type(uint256).max, false);
        console.log("The expectRevert() was successfull, thus it reverted.");
        console.log(
            "As you can see, these are valid operations. And as seen in SCENARIO-1, if signaled with credit they do execute."
        );
        console.log(
            "Note that a 0 profit exit for the exitor makes sense. As maybe Alice had an urgency and suddenly needs money she lent."
        );
        console.log("Thus does not mind exiting her position with 0 profit.");
        console.log("CONCLUSION: This checks also reverts valid exits.");
    }
```

</details>

---

### **Recommended Actions** 🛠️

If the `exactAmountIn` flag signals that `params.amount` is cash, make the check with `minimumCreditBorrowAToken` only after the resulting exact amount of credit has been calculated. The check should compare the exact amount of credit being created or exited with `minimumCreditBorrowAToken`. It could be placed for example after these lines of code: [in buyCredit](https://github.com/code-423n4/2024-06-size/blob/main/src/libraries/actions/BuyCreditMarket.sol#L177), [in sellCredit](https://github.com/code-423n4/2024-06-size/blob/main/src/libraries/actions/SellCreditMarket.sol#L182).

If you prefer to stick to the validate/execute pattern you might calculate reuslting amounts of credit there and keep the new checks there.
