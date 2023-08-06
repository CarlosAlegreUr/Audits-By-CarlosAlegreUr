## Summary 📌

Due to a lack of comments or users' mistakes, users can mistakenly call functions that compute the `loanRatio` with incorrect token amount values, resulting in potential loss of funds.

---

## Vulnerability Details 🔍

### Decimals Handling Issues

When working with tokens that have different decimal precision, the current implementation exhibits inconsistent code which leads to financial losses and unintended behaviors.

<details> <summary> Problem description and analysis 🕵️ </summary>

The main issue arises from the absence of checks on client-provided input. These checks can be performed using the `ERC20.decimals()` function. Unfortunately, due to the custom version of the ERC20 interface used in the project, `IERC20.decimals()` is missing.

The way `loanRatio` is computed leads to contradictory expected inputs for functions that calculate it:

```
uint256 loanRatio = (debt * 10 ** 18) / collateral;
```

Given that token pairs in a pool can have varying decimals, the possible combinations are:

1️⃣ Decimals Debt Token == Decimals Collateral Token

2️⃣ Decimals Debt Token < Decimals Collateral Token

3️⃣ Decimals Debt Token > Decimals Collateral Token

Given the way the `loanRatio` is computed and considering the above scenarios, the expected input values are:

- (==): Both debt and collateral values multiplied by `10^ERC20.decimals()`.
- (<): Desired debt value multiplied by `10^(debt.decimals) * 10^(Difference in Decimals)`. And collateral value multiplied by `10^(collateral.decimals)`.
- (>): Desired debt value multiplied by `10^(debt.decimals)`. And collateral by `10^(collateral.decimals) * 10^(Difference in Decimals)`.

However, these calculations can lead to significant issues, particularly in scenarios 2 and 3. In them if the user sends the correct adjusted values so the `loanRatio` computation makes sense, later, the quanities of tokens transfered won't be the proper ones as they are not readjusted in the code.

Check the `borrow()` function for example:

If debt token has 8 decimals and collateral token has 6 decimals.

```diff
function borrow(Borrow[] calldata borrows) public {
    for (uint256 i = 0; i < borrows.length; i++) {
        bytes32 poolId = borrows[i].poolId;
        // 🟢 Client sends the adjusted values so loanRatio makes sense
        // 🟢 debt = Quantity * 10^8
        uint256 debt = borrows[i].debt;

         // 🟢 collateral = Quantity * 10^6 * 10^(8-6) = Quantity * 10^6 * 10^2
        uint256 collateral = borrows[i].collateral;

        // more code...

        // make sure the user isn't borrowing too much
        uint256 loanRatio = (debt * 10 ** 18) / collateral;
        if (loanRatio > pool.maxLoanRatio) revert RatioTooHigh();
        // create the loan
        Loan memory loan = Loan({
           // 🟢 We are using the adjusted values
            debt: debt,
            collateral: collateral,
        // More loan parameters...
        });

        // more code...

        // transfer the loan tokens from the pool to the borrower
        // 🟢 In this pool debt manages 8 decimals so this transfer is correct
        IERC20(loan.loanToken).transfer(msg.sender, debt - fees);
        // transfer the collateral tokens from the borrower to the contract
        // 🟢 But collateral manages 6 decimals, and collateral has been
        // adjusted for 8 decimals, thus sending 10^2 more tokens than desired.
        IERC20(loan.collateralToken).transferFrom(
            msg.sender,
            address(this),
            collateral // 🟢 <-- The collateral value is used
        );
       // more code...
    }
}
```

**Attack vectors**: pools resembling scenarios 2 and 3. An example of a pool with valuable assets could be:

- Debt : USDT (6 decimals) 💵
- Collateral : WBTC (8 decimals) 💵

...or vice versa.

> 🚧 **Note** ⚠️: Short auction times can exacerbate the vulnerabilities explained along the report.

</details>

### Process Explanation: A "fishing pool" to fish peoples' loans

<details> <summary> Step by step breakdown 🧑‍🔬 </summary>

### 1️⃣ Initiate Loan Request

> 📘 **Notice** ℹ️: Assume the bitcoin price is lower than its nowadays values, making this loan scenario more realistic. The point of this example does not reside exactly in using WBTC and USDT. Rather it resides in using 2 tokens with real value and different decimals.

- The user intends to borrow 10 USDT and deposit 1 WBTC as collateral.
- The `loanRatio` (before considering token decimals) is:

loanRatio = Debt / Collateral = 10 USDT / 1 WBTC = 10

### 2️⃣ Adjust for Token Decimals

As Solidity only handles integer values, we must adjust for decimals to avoid losing precision.

- **USDT**: Has 6 decimals. Therefore, 10 USDT in its smallest unit is:
  10 \* 10^6 = 10,000,000

- **WBTC**: Has 8 decimals. Therefore, 1 WBTC in its smallest unit is:
  1 \* 10^8 = 100,000,000

### 3️⃣ Adjust Decimals for the `loanRatio` calculation

For the `loanRatio` to be calculated correctly, the decimals of the debt and the collateral must match.

- **USDT**: Already has `10^8` (from the previous step).
- **WBTC**: Needs to be adjusted to match the 8 decimals of USDT. This means multiplying by:
  10^(8 - 6) = 10^2 = 100

This gives collateral's vairable
a WBTC quantity of = 1 WBTC \* 100 = **100 WBTC**

### 4️⃣ Potential Risk

The `loanRatio` calculation will now be accurate, since both the debt and collateral are represented in `10^8` units. However, when the user sends the WBTC as collateral, they won't be sending just 1 WBTC, but rather 100 WBTC!

If, for any reason, the user has approved the transfer of more tokens than intended (e.g., due to a generous approval or some unexpected behavior from some scammer leveraging the `_beforeTokenTranser()` of an ERC777 token compatible with ERC20), they could end up depositing much more collateral than necessary. This could lead to potential theft of their tokens or miscomputations.

> 🚧 **Note** ⚠️ : If WBTC were to be the debt, consequences would be the same but the borrower instead of owing 1 WBTC as expected, he would be owing 100 WBTC.

> 🚧 **Caution** ⚠️ : Always ensure that the token adjustments for decimals are clearly communicated to the user and accurately handled in the smart contract to avoid unexpected token transfers. More on this on the **Recommendations** section.

 </details>

<details> <summary> PoC for scenario (debtDecimals > collateralDecimals) 🧑‍🔧 </summary>

Create a new test file in the tests directory, then run the PoC and read the console logs to guide you through the PoC's code.

Run the PoC with:

```
forge test --match-test test_LosingFundToFishingPool -vvv
```

Code 📝:

```
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
import "../src/Lender.sol";

import "forge-std/console.sol";

// import {ERC20} from "solady/src/tokens/ERC20.sol";
import {ERC20DecimalsMock} from "openzeppelin-contracts/contracts/mocks/token/ERC20DecimalsMock.sol";
import {ERC20} from "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";

contract WBTCmock is ERC20DecimalsMock {
    constructor(uint8 decimals_) ERC20DecimalsMock(decimals_) ERC20("Wrapped Bitcoin", "WBTC") {}

    function mint(address account, uint256 amount) public virtual {
        _mint(account, amount);
    }
}

contract USDTmock is ERC20DecimalsMock {
    constructor(uint8 decimals_) ERC20DecimalsMock(decimals_) ERC20("Tether", "USDT") {}

    function mint(address account, uint256 amount) public virtual {
        _mint(account, amount);
    }
}

contract FishingPoolPoC is Test {
    Lender public lender;

    WBTCmock public WBTC_mock;
    USDTmock public USDT_mock;

    bytes32 public poolAsCol;

    address public loser = address(0x1);
    address public fisherman = address(0x2);

    uint256 public constant WBTC_BALANCE = 1000;
    uint256 public constant USDT_BALANCE = 100000;

    function setUp() public {
        lender = new Lender();

        // Using ERC20DecimalsMock by OpenZeppelin
        WBTC_mock = new WBTCmock(8);
        USDT_mock = new USDTmock(6);

        // We mint tokens to both addresses.
        WBTC_mock.mint(address(fisherman), WBTC_BALANCE * 10 ** WBTC_mock.decimals());
        WBTC_mock.mint(address(loser), WBTC_BALANCE * 10 ** WBTC_mock.decimals());

        USDT_mock.mint(address(loser), USDT_BALANCE * 10 ** 18);
        USDT_mock.mint(address(fisherman), USDT_BALANCE * 10 ** 18);

        // You end up loosing the token with smaller decimal size, in this case WBTC
        // Pool Example, lost as collateral => USDT, Borrow => WBTC
        createPoolWBTCAsCollatral(fisherman);
    }

    function createPoolWBTCAsCollatral(address l) private returns (Pool memory p) {
        vm.startPrank(l);
        USDT_mock.approve(address(lender), USDT_BALANCE * 10 ** USDT_mock.decimals());
        p = Pool({
            lender: l,
            loanToken: address(USDT_mock),
            collateralToken: address(WBTC_mock),
            minLoanSize: 10 * 10 ** USDT_mock.decimals(),
            poolBalance: 1000 * 10 ** WBTC_mock.decimals(),
            maxLoanRatio: 15 * 10 ** 18,
            auctionLength: 1,
            interestRate: 1000,
            outstandingLoans: 0
        });
        poolAsCol = lender.setPool(p);
        vm.stopPrank();
    }

    function test_LosingFundToFishingPool() public {
        console.log("------ PoC: Losing Funds to Fishing Pool ------");

        console.log("Initiating borrow of 10 USDT in exchange for 1 WBTC as collateral...");

        uint256 wbtcBackToIntFactor = 10 ** WBTC_mock.decimals();
        uint256 usdtBackToIntFactor = 10 ** USDT_mock.decimals();
        uint256 balance = WBTC_mock.balanceOf(loser) / wbtcBackToIntFactor;

        console.log("Initial WBTC balance for borrower: %s WBTC", balance);
        console.log("Expected 1 WBTC increase post-borrow.");

        uint256 usdtBalance = USDT_mock.balanceOf(loser) / usdtBackToIntFactor;
        console.log("Initial USDT balance for borrower: %s USDT", usdtBalance);
        console.log("Expected 1 USDT increase post-borrow.");

        uint8 adjustedDecimals = WBTC_mock.decimals() - USDT_mock.decimals();
        vm.startPrank(loser);
        WBTC_mock.approve(address(lender), 1000);

        console.log("Given maxLoanRatio of the pool is 15, borrowing should occur...");
        uint256 usdtToBorrow = 10 * 10 ** USDT_mock.decimals();
        uint256 btcCollateral = 1 * 10 ** WBTC_mock.decimals() * 10 ** adjustedDecimals;
        console.log("We relaize loanRatio is calculated only adjusting for PIBs and not for tokens decimals.");
        console.log("So we adjust them.");

        Borrow memory b = Borrow({poolId: poolAsCol, debt: usdtToBorrow, collateral: btcCollateral});
        Borrow[] memory borrows = new Borrow[](1);
        borrows[0] = b;

        console.log("Attempting a valid borrow. There shouldn't be a revert...");
        vm.expectRevert();
        lender.borrow(borrows);
        console.log("Revert detected. This shouldn't happen...");

        console.log(
            "We realize if we just call the function with each token adjusted to it's own decimals, loanRatio check will pass."
        );
        console.log("Retrying borrow without adjusting the difference in decimals of tokens");
        usdtToBorrow = 10 * 10 ** USDT_mock.decimals();
        WBTC_mock.approve(address(lender), WBTC_BALANCE * 10 ** WBTC_mock.decimals());
        b = Borrow({poolId: poolAsCol, debt: usdtToBorrow, collateral: btcCollateral});
        borrows = new Borrow[](1);
        borrows[0] = b;
        lender.borrow(borrows);
        console.log("Borrow successfull");
        balance = WBTC_mock.balanceOf(loser) / wbtcBackToIntFactor ;
        console.log("Post-borrow WBTC balance for borrower: %s WBTC", balance);
        console.log("Yes, it's 900, we wanted to give 1 and we gave 100.");
        console.log("Unexpected WBTC loss detected. Lender can potentially end up owning collateral via auction.");

        usdtBalance = USDT_mock.balanceOf(loser) / usdtBackToIntFactor;
        console.log("Post-borrow USDT balance for borrower: %s USDT", usdtBalance);
        console.log("USDT increment observed as expected.");

        vm.stopPrank();
        console.log("------ PoC Complete ------");
    }
}
```

</details>

> 📘 **Notice** ℹ️: PoC stands for "Proof Of Concept", the minimum amount of code to prove your statement.

---

## Impact 📈

- **Core Concern** 🔻: The contract inconsistently manages token decimals, resulting in a misleading API prone to user errors. Rather than relying on users' vigilance, smart contracts should inherently reduce potential mistakes, especially with financial implications.

- **Risks** 🥷: Beyond financial losses, scammers can exploit these inconsistencies.

In essence, this vulnerability is not just a potential drain on users' wallets but a strain on trust in the system.

---

## Tools Used 🛠️

- Manual audit
- Slither
- The tests created for the PoC.

---

## Recommendations 🎯

### 1️⃣ **User-Friendly API Design**:

- **Objective**: Simplify the API to accept intuitive values for debt and collateral. Then make the contract internally adjust decimals. Though this might increase gas costs, it reduces potential financial errors, enhancing the protocol's accessibility. Of course, users can still accidentally send more money than intended, but this time the confusion doesn't stem from the system's API.

  To simplify the users' exeprience with the API you can:

  - 1️⃣ Refactor the `Borrow` struct by adding `collateralAdjusted` and `debtAdjusted`, calculated as `value * 10^ERC20.decimals()`.

  - 2️⃣ **Developer Notes**: For clarity, prepend in the Borrow structure a comment about this:

    ```
    @dev Decimals In Tokens ⚠️: adjustedCollateral and adjusteDebt are the token quantities adapted to their ERC20 decimals:
    tokenQuantity * 10 ^ (ERC20.decimals())
    ```

  - 3️⃣ Apply a conversion ratio for a precise `loanRatio` computation and integrate checks to validate transfer amounts.

    > 📘 **Notice** ℹ️ These extra parameters in the Borrow struct are what enables the transfered amounts checks.

<details> <summary> New System's Logic Integration Example ⚙️ </summary>

Due to the for-loops in the code, we will use private functions instead of modifiers

> 🚧 **Note** ⚠️: The functions shown are illustrative and untested.

**_`New ratio computation`_**

```
function computeRatio(uint256 debt, address debtAddress, uint256 collateral, address collateralAddress) private pure returns (uint256) {
    // 🟢 Prepare values for adjustments computations
    uint256 adjustment;
    uint256 debtDecimals = IERC20(debtAddress).decimals();
    uint256 collateralDecimals = IERC20(collateralAddress).decimals();

    // 🟢 Detect and adjust the value with the smallest decimals
    if (debtDecimals > collateralDecimals) {
        adjustment = 10 ** (debtDecimals - collateralDecimals);
        debt *= adjustment * debtDecimals;
    } else if (collateralDecimals > debtDecimals) {
        adjustment = 10 ** (collateralDecimals - debtDecimals);
        collateral *= adjustment * collateralDecimals;
    } else {
        // Notice as this line will execute when decimals are equal, doesn't matter if using
        // collateral decimals or debt ones.
        adjustment = 10 ** debtDecimals;
    }

    // 🟢 Now the previous formula will be valid
    uint256 loanRatio = (debt * 10 ** 18) / collateral;
    return loanRatio;
}
```

**_`Quantities Checking`_**

```
function checkQuantitiesFormat(uint256 debt, uint256 adjustedDebt, uint256 collateral, uint256 adjustedCollateral, address debtAddress, address collateralAddress) private pure {
    IERC20 collateralTkn = IERC20(collateralAddress).decimals();
    uint256 collateralDecimalsFactor = 10 ** collateralTkn.decmials();

    // 🟢 User must provide both formats and here we check if those formats
    // represent the same value.
    bool validCollateralFormat = ((collateral * collateralDecimalsFactor) == adjustedCollateral);
    if(!validCollateralFormat){
      revert CollateralQuantitesDiffer();
    }

    // 🟢 Same with collateral values.
    IERC20 debtTkn = IERC20(debtAddress);
    uint256 debtDecimalsFactor = 10 ** debtTkn.decmials();
    bool validDebtFormat = ((debt * debtDecimalsFactor) == adjustedDebt);
    if(!validDebtFormat){
      revert DebtQuantitesDiffer();
    }
}
```

</details>

### 2️⃣ **Integration with IERC20Metadata.sol**:

Adapt the `IERC20` to use the `decimals()` method or, preferably use [IERC20Metadata.sol](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20Metadata) from OpenZeppelin which provides the `decimals()` function.

> 🚧 **Note** ⚠️: Please, take another audit after fixing this issues.
