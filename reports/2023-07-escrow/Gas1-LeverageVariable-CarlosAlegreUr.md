### Summary 📌

All tests pass after the gas optimization, total gas saved was **2.275%**.

The main optimization strategy involves replacing the following expression:
`i_tokenContract.balanceOf(address(this))`
with the immutable variable `i_price`. They should ideally have the same value.

This change enhances efficiency as Solidity checks an immutable value directly, avoiding the need to invoke an external function in a third-party contract.

### Vulnerability Details 🔍

Two functions were modified:

1. _**`resolveDispute()`**_

<details> <summary> Before & After 🌃🌇 </summary>

#### Before

```
   function resolveDispute(uint256 buyerAward) external onlyArbiter nonReentrant inState(State.Disputed) {
       uint256 tokenBalance = i_tokenContract.balanceOf(address(this));
       uint256 totalFee = buyerAward + i_arbiterFee;
       if (totalFee > tokenBalance) {
           revert Escrow__TotalFeeExceedsBalance(tokenBalance, totalFee);
       }
       _;
       // ... (unchanged code)
       tokenBalance = i_tokenContract.balanceOf(address(this));
       if (tokenBalance > 0) {
           i_tokenContract.safeTransfer(i_seller, tokenBalance);
       }
   }
```

#### After

```
function resolveDispute(uint256 buyerAward) external onlyArbiter nonReentrant inState(State.Disputed) {
    // Here!🟢 notice tokenBalance not set neither declared here anymore
    uint256 totalFee = buyerAward + i_arbiterFee;
    if (totalFee > i_price) {
        revert Escrow__TotalFeeExceedsBalance(i_price, totalFee);
    }
    _;
    // ... (unchanged code)
    // tokenBalance set here!! 🟢
    uint256 tokenBalance = i_tokenContract.balanceOf(address(this));
    if (tokenBalance > 0) {
        i_tokenContract.safeTransfer(i_seller, tokenBalance);
    }
}
```

</details>

2. _**`confirmReceipt()`**_

<details> <summary> Before & After 🌃🌇 </summary>

At **_`line 98`_**.

#### Before

```
   i_tokenContract.safeTransfer(i_seller, i_tokenContract.balanceOf(address(this)));
```

#### After

```
   i_tokenContract.safeTransfer(i_seller, i_price);
```

</details>

### Impact 📈

The impact has been measured in new isolated tests and in the client's tests that involved calls which executed the changes applied.

The total gas saved is **2.275%**.

| _Test Type_      | _Optimized Gas_ | _Original Gas_ | _Difference_ | _% Saved on Original Gas_ |
| ---------------- | --------------- | -------------- | ------------ | ------------------------- |
| **All tests**    | 9,262,601       | 9,477,939      | 215,338      | **2.27%**                 |
| **Personalized** | 1,525,223       | 1,561,105      | 35,882       | **2.29%**                 |
| **Total**        | 10,787,824      | 11,039,044     | 251,220      | **2.275%**                |

#### Details:

<details> <summary> Isolated Tests Used 🧪 </summary>

_**`Tests`**_

This test focuses on the `resolveDispute()` execution costs:

```
function testResolveDisputeGasConsumption(uint256 callLotsOfTimes) public escrowDeployed {
    vm.prank(BUYER);
    escrow.initiateDispute();
    vm.prank(ARBITER);
    escrow.resolveDispute(buyerAward);
}
```

And this one focuses on the `confirmReceipt()`'s ones:

```
function testConfirmReceiptGasConsumption(uint256 callLotsOfTimes) public escrowDeployed {
    vm.prank(BUYER);
    escrow.confirmReceipt();
}
```

The tests were executed with 256 fuzz runs.

| Function Name      | Percentage Saved |
| ------------------ | ---------------- |
| `confirmReceipt()` | 2.34%            |
| `resolveDispute()` | 2.25%            |

 </details>

 <details> <summary> Test by test breakdown 🧑‍🔬 </summary>

These are all the tests used in the analysis.

> 🚧 **Notice** ⚠️: Some tests that involved checking for reverts have been excluded from the analysis. This is because they didn't execute the optimized code, thus its gas consumption won't be affected by the changes.

| _Test Name_                                | _Optimized Gas_ | _Original Gas_ | _Gas Saved_ |
| ------------------------------------------ | --------------- | -------------- | ----------- |
| testConfirmReceiptGasConsumption           | 746,542         | 764,468        | 17,926      |
| testResolveDisputeGasConsumption           | 778,681         | 796,637        | 17,956      |
| testResolveDisputeChangesState             | 779,946         | 797,902        | 17,956      |
| testCanOnlyResolveInDisputedState          | 756,577         | 774,521        | 17,944      |
| testConfirmReceiptEmitsEvent               | 747,825         | 765,751        | 17,926      |
| testResolveDisputeWithBuyerAward           | 830,930         | 848,886        | 17,956      |
| testStateChangesOnConfirmedReceipt         | 747,763         | 765,689        | 17,926      |
| testResolveDisputeZeroArbiterFeeTransfer   | 760,144         | 778,100        | 17,956      |
| testResolveDisputeTransfersTokens          | 830,506         | 848,462        | 17,956      |
| testTransfersTokenOutOfContract            | 748,383         | 766,309        | 17,926      |
| testCanOnlyInitiateDisputeInConfirmedState | 748,501         | 766,427        | 17,926      |
| testResolveDisputeZeroBuyerTransfer        | 783,852         | 801,808        | 17,956      |
| testResolveDisputeZeroSellerTransfer       | 784,268         | 802,225        | 17,957      |
| testResolveDisputeFeeExceedsBalance        | 743,906         | 761,859        | 17,953      |
| **TOTAL**                                  | 10,787,824      | 11,039,044     | 251,220     |

Total saved percentage => **2.275%**.

> 📘 **Notice** ℹ️: The percentage has been calculated with these numbers from the TOTAL:
>
> (251,220 / 11,039,044) \* 100
>
> They mean:
>
> (totalGasSaved / originalGasCost) \* 100

 </details>

 <details> <summary> Forge Snapshots from isolated test 📸 </summary>

_**`Original`**_

```
    EscrowTestGas:testConfirmReceiptGasConsumption(uint256) (runs: 256, μ: 764468, ~: 764468)
    EscrowTestGas:testResolveDisputeGasConsumption(uint256) (runs: 256, μ: 796637, ~: 796637)
```

_**`Optimized`**_

```
    EscrowTestGas:testConfirmReceiptGasConsumption(uint256) (runs: 256, μ: 746542, ~: 746542)
    EscrowTestGas:testResolveDisputeGasConsumption(uint256) (runs: 256, μ: 778681, ~: 778681)
```

 </details>

 <details> <summary> Forge Snapshots from client's test 📸 </summary>

_**`Original`**_

```
    EscrowTest:testTransfersTokenOutOfContract() (gas: 748383)
    EscrowTest:testStateChangesOnConfirmedReceipt() (gas: 747763)
    EscrowTest:testConfirmReceiptEmitsEvent() (gas: 747825)
    EscrowTest:testCanOnlyInitiateDisputeInConfirmedState() (gas: 748501)
    EscrowTest:testCanOnlyResolveInDisputedState() (gas: 756577)
    EscrowTest:testResolveDisputeChangesState() (gas: 779946)
    EscrowTest:testResolveDisputeTransfersTokens() (gas: 830506)
    EscrowTest:testResolveDisputeFeeExceedsBalance() (gas: 743906)
    EscrowTest:testResolveDisputeWithBuyerAward() (gas: 830930)
    EscrowTest:testResolveDisputeZeroSellerTransfer() (gas: 784268)
    EscrowTest:testResolveDisputeZeroBuyerTransfer() (gas: 783852)
    EscrowTest:testResolveDisputeZeroArbiterFeeTransfer() (gas: 760144)
```

_**`Optimized`**_

```
    EscrowTest:testTransfersTokenOutOfContract() (gas: 766309)
    EscrowTest:testStateChangesOnConfirmedReceipt() (gas: 765689)
    EscrowTest:testConfirmReceiptEmitsEvent() (gas: 765751)
    EscrowTest:testCanOnlyInitiateDisputeInConfirmedState() (gas: 766427)
    EscrowTest:testCanOnlyResolveInDisputedState() (gas: 774521)
    EscrowTest:testResolveDisputeChangesState() (gas: 797902)
    EscrowTest:testResolveDisputeTransfersTokens() (gas: 848462)
    EscrowTest:testResolveDisputeFeeExceedsBalance() (gas: 761859)
    EscrowTest:testResolveDisputeWithBuyerAward() (gas: 848886)
    EscrowTest:testResolveDisputeZeroSellerTransfer() (gas: 802225)
    EscrowTest:testResolveDisputeZeroBuyerTransfer() (gas: 801808)
    EscrowTest:testResolveDisputeZeroArbiterFeeTransfer() (gas: 778100)
```

</details>

### Tools Used 🛠️

- Manual auditing.
- Forge snapshot.
- Bash scripts tailored to analyze forge snapshots.

> 📘 **Notice** ℹ️: I've personally created the bash scripts. Here is a link to the github repo [Forge-Snapshots-Analyzer](https://github.com/CarlosAlegreUr/Forge-Snapshots-Analyzer).

### Recommendations 🎯

While the `balanceOf()` method might enhance readability, developers should decide to balance between readability and efficiency. A proposed way to achieve both is by adding a comment above the `i_price` variable, indicating its intended purpose and expected behavior:

```
    // This variable should always match: i_tokenContract.balanceOf(address(this))
    uint256 private immutable i_price;
```
