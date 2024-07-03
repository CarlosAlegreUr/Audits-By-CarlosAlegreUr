
# Medium Risk Issue: `Size` 🕵️

---

# **Users can skip the fragmentation fee through the `compensate()` action.**

---

### **Explanation && Impact** 📌📈

Users can skip the fragmentation fee through the `compensate()` action.

The impact is that those fees are needed to keep the bots of the protocol up and running plus the following invariant from the code is broken:

```text
FEES_01: Fragmentation fees are applied whenever there is a credit fractionalization
```

To avoid the framentation fee you have to call `compensate()` with the following inputs:

- `creditPositionWithDebtToRepayId` == Any position where you are borrower and want to fragmentate for free
- `creditPositionToCompensateId` == RESERVED_ID
- `amount` == whatever amount you want to fragmentate it into, bigger than `minimumCreditBorrowAToken`.

This happens because of skipping [this if statement](https://github.com/code-423n4/2024-06-size/blob/main/src/libraries/actions/Compensate.sol#L146). Follow the comments on the code for more details:

```solidity
        CreditPosition memory creditPositionToCompensate;
        if (params.creditPositionToCompensateId == RESERVED_ID) {
            creditPositionToCompensate = state.createDebtAndCreditPositions({
                lender: msg.sender,
                borrower: msg.sender,
                futureValue: amountToCompensate, // 🟢1️⃣ Creates `creditPositionToCompensate` with `amountToCompensate` credit
                dueDate: debtPositionToRepay.dueDate
            });
        } else { /* doesnt matter, will not execute */ }

        // debt and credit reduction
        state.reduceDebtAndCredit( // 🟢2️⃣ Notice this does not alter `creditPositionToCompensate` in any way
            creditPositionWithDebtToRepay.debtPositionId, params.creditPositionWithDebtToRepayId, amountToCompensate
        );
        // 🟢3️⃣ Deducts `amountToCompensate` from `creditPositionToCompensate.credit`
        // Which as seen in step 1️⃣, they are the same amount. Thus result is 0.
        uint256 exiterCreditRemaining = creditPositionToCompensate.credit - amountToCompensate;

        // credit emission
        // 🟢4️⃣ As we passed RESERVED_ID, the last position created gets exited, and as `amountToCompensate`
        // is its full amount. `createCreditPosition()` will just change the lender. So now we have 2 positions
        // with credit: `creditPositionWithDebtToRepay` that has been partially repayed and `creditPositionToCompensate`
        // with `amountToCompensate` amount of credit.
        state.createCreditPosition({
            exitCreditPositionId: params.creditPositionToCompensateId == RESERVED_ID
                ? state.data.nextCreditPositionId - 1
                : params.creditPositionToCompensateId,
            lender: creditPositionWithDebtToRepay.lender,
            credit: amountToCompensate
        });
        // 🟢5️⃣ As we saw on 3️⃣, this is 0 so code inside won't execute. But we have created 2 different positions of msg.sender
        // with the same lender that will have to be claimed at some point. Credit has been fractionalized.
        if (exiterCreditRemaining > 0) {
            // Code charges frag fee inside here
        }
        // end of function
```

---

### **Proof Of Concept (PoC)** 👨‍💻💻

Paste the code below it in some test file `(./test/local/actions/*.t.sol)` on the system, import foundry _console.log_ in that file: `import "forge-std/console.sol";`. And run:

```bash
forge test --match-test "test_avoidFragFee" -vvv
```

<details> <summary> See code 👁️ </summary>

> ℹ️ **Note** 📘 If the test file you paste this test in requires more imports please read linter warnings about missing imports and add them.

```solidity
    function test_avoidFragFee() public {
        _deposit(alice, weth, 500e18);
        _deposit(alice, usdc, 500e6);
        _deposit(bob, weth, 500e18);
        _deposit(bob, usdc, 500e6);

        uint256[] memory tenors = new uint256[](2);
        tenors[0] = 1 days;
        tenors[1] = 2 days;
        int256[] memory aprs = new int256[](2);
        aprs[0] = 1.01e18;
        aprs[1] = 1.02e18;
        uint256[] memory marketRateMultipliers = new uint256[](2);
        console.log("Alice lends money to Bob.");
        console.log(
            "Bob will split his position into 2 without paying fragmentation fee, for that he does comepnsate(RESERVED_ID)."
        );

        // Bob puts a borrow offer, then Alice calls buyCreditMarket
        _sellCreditLimit(bob, YieldCurve({tenors: tenors, aprs: aprs, marketRateMultipliers: marketRateMultipliers}));
        uint256 amount = 100e6; // 100 credit
        uint256 tenor = 2 days;
        _buyCreditMarket(alice, bob, amount, tenor, false);
        (, uint256 creditCount) = size.getPositionsCount();
        uint256 creditPositionId = CREDIT_POSITION_ID_START + creditCount - 1;

        Vars memory v = _state();
        console.log("Fees before compensate: ", v.feeRecipient.collateralTokenBalance);
        _compensate(bob, creditPositionId, RESERVED_ID, amount / 2);

        v = _state();
        console.log("Fees after compensate: ", v.feeRecipient.collateralTokenBalance);
        (, creditCount) = size.getPositionsCount();
        uint256 lastCreatedId = CREDIT_POSITION_ID_START + creditCount - 1;

        console.log("ID of position created during compensate: ", lastCreatedId);
        console.log("Credit of position created during compensate: ", size.getCreditPosition(lastCreatedId).credit);
        console.log("ID of position created with Alice:", creditPositionId);
        console.log("Credit of position created with Alice: ", size.getCreditPosition(creditPositionId).credit);
        console.log("As you can see, 2 different credit positions, both with some credit, have been created.");
        console.log("But no fragmentation fee has arrved to feeReceiver.");
        console.log("IMPACT: Users have a way to fragmentate credit avoiding feragmentaiton fee via the compensate function.");
    }
```

</details>

---

### **Recommended Actions** 🛠️

Check at the end of execution for the credit amount of all positions created or manipulated along the `compensate` action. If any of them has credit 0 it means that there was no fragmentation, if both have credit > 0, it means there was fragmentation.

Something similar to:

> ⚠️ **WARNING** 🚧 The code below is not audited. It is just a template to guide devs. The key takeaway of the example is to realize that the fragmentation check must be done differently when passing `RESERVED_ID`.

```solidity
bool frag = false;

uint256 creditOfPositionUsed = params.creditPositionToCompensateId == RESERVED_ID ? getCreditPosition(state.data.nextCreditPositionId - 1).credit : getCreditPosition(params.creditPositionToCompensateId).credit;

if(creditOfPositionUsed != 0 && getCreditPosition(creditPositionWithDebtToRepay).credit != 0){
    frag = true; 
}

if(frag){
    //charge fragmentation fee...
}
```
