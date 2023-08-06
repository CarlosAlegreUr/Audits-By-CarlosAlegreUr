## 📌 Summary

After this optimization, client-tests still pass, with gas savings of **0.217%**. Because of being a compiling process optimization, it has been applied to all contracts in scope.

---

## 🔍 Vulnerability Details

### 1️⃣ Use `via_ir = true` for an Enhanced Compiler Process

The `via_ir` flag in the `foundry.toml` file triggers a more optimized compiler. This change resulted in a **0.217%** improvement over the unoptimized contract.

**`foundry.toml` Configuration**:

```
(options...)
remappings = [...]
via_ir = true  <-- Set flag for optimization
```

> 🚧 **Note** ⚠️: Enabling `via_ir` makes the compilation process slower. Compilation-related tasks are expected to take quite a bit more of time when this flag is active.

> 📘 **Information** ℹ️: The `via_ir` flag instructs Foundry to utilize Intermediate Representation (IR). IR is a low-level language bridging high-level source code and machine code. This intermediate step allows for additional optimization techniques. That's what the `_ir` means in `via_ir`, Intermediate Representation..

---

## 📈 Impact

Metrics derived from gas consumption of the provided tests:

| Optimization Method | Optimized Gas | Original Gas | Gas Saved | % Saved |
| ------------------- | :-----------: | :----------: | :-------: | :-----: |
| **Use of via_ir**   |  15,039,663   |  15,072,846  |  33,183   | 0.217%  |

Metrics were taken by running the `forge snapshot` command and compared with bash scripts tailored to analyze the results of forge snapshots.

<details> <summary> Test by test breakdown 🧑‍🔬 </summary>

> 🚧 **Notice** ⚠️: The optimization affects all the contract, so I've analyzed all the tests but invariant ones because `forge snapshot` does not provide gas consumtion metrics for those.

> 🚧 **Notice** ⚠️: Negative numbers in the `Gas Saved` column means the "optimization" increased the consumption of gas in that test. But the overall outcome in this case is positive so it can be considered an optimization.

| Test Name                                                   | Optimized Gas | Original Gas | Gas Saved |
| ----------------------------------------------------------- | ------------- | ------------ | --------- |
| DSCEngineTest:testCanMintWithDepositedCollateral            | 202,684       | 203,164      | 480       |
| DSCEngineTest:testHealthFactorCanGoBelowOne                 | 287,373       | 290,012      | 2,639     |
| DSCEngineTest:testGetCollateralBalanceOfUser                | 83,913        | 83,444       | -469      |
| DSCEngineTest:testGetUsdValue                               | 26,044        | 26,254       | 210       |
| DSCEngineTest:testGetAccountCollateralValue                 | 126,014       | 127,827      | 1,813     |
| DSCEngineTest:testRevertsWithUnapprovedCollateral           | 708,487       | 707,851      | -636      |
| DSCEngineTest:testLiquidatorTakesOnUsersDebt                | 474,882       | 483,522      | 8,640     |
| DSCEngineTest:testUserStillHasSomeEthAfterLiquidation       | 488,630       | 498,431      | 9,801     |
| DSCEngineTest:testCantLiquidateGoodHealthFactor             | 349,925       | 354,163      | 4,238     |
| DSCEngineTest:testMustRedeemMoreThanZero                    | 234,352       | 236,959      | 2,607     |
| DSCEngineTest:testRevertsIfTransferFails                    | 1,946,641     | 1,930,291    | -16,350   |
| DSCEngineTest:testCantBurnMoreThanUserHas                   | 14,399        | 13,312       | -1,087    |
| DSCEngineTest:testRevertsIfTokenLengthDoesntMatchPriceFeeds | 181,848       | 180,675      | -1,173    |
| DSCEngineTest:testMustImproveHealthFactorOnLiquidation      | 2,364,258     | 2,363,648    | -610      |
| DSCEngineTest:testGetLiquidationThreshold                   | 5,880         | 5,616        | -264      |
| DSCEngineTest:testGetCollateralTokenPriceFeed               | 12,251        | 12,268       | 17        |
| DSCEngineTest:testCanRedeemDepositedCollateral              | 248,461       | 251,655      | 3,194     |
| DSCEngineTest:testRevertsIfMintAmountIsZero                 | 199,612       | 201,106      | 1,494     |
| DSCEngineTest:testUserHasNoMoreDebt                         | 475,044       | 483,424      | 8,380     |
| DSCEngineTest:testCanRedeemCollateral                       | 127,418       | 127,928      | 510       |
| DSCEngineTest:testProperlyReportsHealthFactor               | 207,663       | 210,229      | 2,566     |
| DSCEngineTest:testGetTokenAmountFromUsd                     | 26,493        | 26,170       | -323      |
| DSCEngineTest:testCanDepositedCollateralAndGetAccountInfo   | 128,518       | 130,228      | 1,710     |
| DSCEngineTest:testRevertsIfTransferFromFails                | 1,967,717     | 1,955,875    | -11,842   |
| DecentralizedStablecoinTest:testCantMintToZeroAddress       | 12,893        | 12,443       | -450      |
| DSCEngineTest:testPriceRevertsOnStaleCheck                  | 21,836        | 21,767       | -69       |
| DSCEngineTest:testGetDsc                                    | 9,041         | 7,794        | -1,247    |
| DSCEngineTest:testCanBurnDsc                                | 234,462       | 237,295      | 2,833     |
| DSCEngineTest:testCanMintDsc                                | 204,032       | 204,291      | 259       |
| DSCEngineTest:testPriceRevertsOnBadAnsweredInRound          | 24,279        | 24,369       | 90        |
| DSCEngineTest:testRevertsIfMintedDscBreaksHealthFactor      | 163,893       | 165,932      | 2,039     |
| DSCEngineTest:testRevertsIfMintFails                        | 2,026,539     | 2,032,604    | 6,065     |
| DSCEngineTest:testRevertsIfBurnAmountIsZero                 | 200,131       | 201,181      | 1,050     |
| DecentralizedStablecoinTest:testMustBurnMoreThanZero        | 60,193        | 59,972       | -221      |
| DecentralizedStablecoinTest:testMustMintMoreThanZero        | 12,380        | 12,062       | -318      |
| DSCEngineTest:testGetMinHealthFactor                        | 6,752         | 5,547        | -1,205    |
| DecentralizedStablecoinTest:testCantBurnMoreThanYouHave     | 60,278        | 59,986       | -292      |
| DSCEngineTest:testRevertsIfCollateralZero                   | 44,899        | 43,623       | -1,276    |
| DSCEngineTest:testCanDepositCollateralWithoutMinting        | 89,157        | 88,270       | -887      |
| DSCEngineTest:testLiquidationPayoutIsCorrect                | 476,403       | 484,401      | 7,998     |
| DSCEngineTest:testGetAccountCollateralValueFromInformation  | 128,558       | 130,226      | 1,668     |
| DSCEngineTest:testGetTimeout                                | 5,605         | 5,532        | -73       |
| DSCEngineTest:testGetCollateralTokens                       | 16,802        | 15,603       | -1,199    |
| **TOTAL**                                                   | 15,052,043    | 15,084,908   | 32,865    |

Total saved percentage => **0.217%**.

> 📘 **Notice** ℹ️: The percentage has been calculated with these numbers from the TOTAL:
>
> ( 32,865 / 15,084,908) \* 100
>
> They mean:
>
> (totalGasSaved / originalGasCost) \* 100

 </details>

<details> <summary> Forge Snapshots Used 📸 </summary>

_**`Original`**_

```
DSCEngineTest:testCanBurnDsc() (gas: 237295)
DSCEngineTest:testCanDepositCollateralWithoutMinting() (gas: 88270)
DSCEngineTest:testCanDepositedCollateralAndGetAccountInfo() (gas: 130228)
DSCEngineTest:testCanMintDsc() (gas: 204291)
DSCEngineTest:testCanMintWithDepositedCollateral() (gas: 203164)
DSCEngineTest:testCanRedeemCollateral() (gas: 127928)
DSCEngineTest:testCanRedeemDepositedCollateral() (gas: 251655)
DSCEngineTest:testCantBurnMoreThanUserHas() (gas: 13312)
DSCEngineTest:testCantLiquidateGoodHealthFactor() (gas: 354163)
DSCEngineTest:testGetAccountCollateralValue() (gas: 127827)
DSCEngineTest:testGetAccountCollateralValueFromInformation() (gas: 130226)
DSCEngineTest:testGetCollateralBalanceOfUser() (gas: 83444)
DSCEngineTest:testGetCollateralTokenPriceFeed() (gas: 12268)
DSCEngineTest:testGetCollateralTokens() (gas: 15603)
DSCEngineTest:testGetDsc() (gas: 7794)
DSCEngineTest:testGetLiquidationThreshold() (gas: 5616)
DSCEngineTest:testGetMinHealthFactor() (gas: 5547)
DSCEngineTest:testGetTimeout() (gas: 5532)
DSCEngineTest:testGetTokenAmountFromUsd() (gas: 26170)
DSCEngineTest:testGetUsdValue() (gas: 26254)
DSCEngineTest:testHealthFactorCanGoBelowOne() (gas: 290012)
DSCEngineTest:testLiquidationPayoutIsCorrect() (gas: 484401)
DSCEngineTest:testLiquidatorTakesOnUsersDebt() (gas: 483522)
DSCEngineTest:testMustImproveHealthFactorOnLiquidation() (gas: 2363648)
DSCEngineTest:testMustRedeemMoreThanZero() (gas: 236959)
DSCEngineTest:testPriceRevertsOnBadAnsweredInRound() (gas: 24369)
DSCEngineTest:testPriceRevertsOnStaleCheck() (gas: 21767)
DSCEngineTest:testProperlyReportsHealthFactor() (gas: 210229)
DSCEngineTest:testRevertsIfBurnAmountIsZero() (gas: 201181)
DSCEngineTest:testRevertsIfCollateralZero() (gas: 43623)
DSCEngineTest:testRevertsIfMintAmountBreaksHealthFactor() (gas: 166563)
DSCEngineTest:testRevertsIfMintAmountIsZero() (gas: 201106)
DSCEngineTest:testRevertsIfMintFails() (gas: 2032604)
DSCEngineTest:testRevertsIfMintedDscBreaksHealthFactor() (gas: 165932)
DSCEngineTest:testRevertsIfRedeemAmountIsZero() (gas: 201395)
DSCEngineTest:testRevertsIfTokenLengthDoesntMatchPriceFeeds() (gas: 180675)
DSCEngineTest:testRevertsIfTransferFails() (gas: 1930291)
DSCEngineTest:testRevertsIfTransferFromFails() (gas: 1955875)
DSCEngineTest:testRevertsWithUnapprovedCollateral() (gas: 707851)
DSCEngineTest:testUserHasNoMoreDebt() (gas: 483424)
DSCEngineTest:testUserStillHasSomeEthAfterLiquidation() (gas: 498431)
DecentralizedStablecoinTest:testCantBurnMoreThanYouHave() (gas: 59986)
DecentralizedStablecoinTest:testCantMintToZeroAddress() (gas: 12443)
DecentralizedStablecoinTest:testMustBurnMoreThanZero() (gas: 59972)
DecentralizedStablecoinTest:testMustMintMoreThanZero() (gas: 12062)
```

_**`Optimized`**_

```
DSCEngineTest:testCanBurnDsc() (gas: 234462)
DSCEngineTest:testCanDepositCollateralWithoutMinting() (gas: 89157)
DSCEngineTest:testCanDepositedCollateralAndGetAccountInfo() (gas: 128518)
DSCEngineTest:testCanMintDsc() (gas: 204032)
DSCEngineTest:testCanMintWithDepositedCollateral() (gas: 202684)
DSCEngineTest:testCanRedeemCollateral() (gas: 127418)
DSCEngineTest:testCanRedeemDepositedCollateral() (gas: 248461)
DSCEngineTest:testCantBurnMoreThanUserHas() (gas: 14399)
DSCEngineTest:testCantLiquidateGoodHealthFactor() (gas: 349925)
DSCEngineTest:testGetAccountCollateralValue() (gas: 126014)
DSCEngineTest:testGetAccountCollateralValueFromInformation() (gas: 128558)
DSCEngineTest:testGetCollateralBalanceOfUser() (gas: 83913)
DSCEngineTest:testGetCollateralTokenPriceFeed() (gas: 12251)
DSCEngineTest:testGetCollateralTokens() (gas: 16802)
DSCEngineTest:testGetDsc() (gas: 9041)
DSCEngineTest:testGetLiquidationThreshold() (gas: 5880)
DSCEngineTest:testGetMinHealthFactor() (gas: 6752)
DSCEngineTest:testGetTimeout() (gas: 5605)
DSCEngineTest:testGetTokenAmountFromUsd() (gas: 26493)
DSCEngineTest:testGetUsdValue() (gas: 26044)
DSCEngineTest:testHealthFactorCanGoBelowOne() (gas: 287373)
DSCEngineTest:testLiquidationPayoutIsCorrect() (gas: 476403)
DSCEngineTest:testLiquidatorTakesOnUsersDebt() (gas: 474882)
DSCEngineTest:testMustImproveHealthFactorOnLiquidation() (gas: 2364258)
DSCEngineTest:testMustRedeemMoreThanZero() (gas: 234352)
DSCEngineTest:testPriceRevertsOnBadAnsweredInRound() (gas: 24279)
DSCEngineTest:testPriceRevertsOnStaleCheck() (gas: 21836)
DSCEngineTest:testProperlyReportsHealthFactor() (gas: 207663)
DSCEngineTest:testRevertsIfBurnAmountIsZero() (gas: 200131)
DSCEngineTest:testRevertsIfCollateralZero() (gas: 44899)
DSCEngineTest:testRevertsIfMintAmountBreaksHealthFactor() (gas: 165142)
DSCEngineTest:testRevertsIfMintAmountIsZero() (gas: 199612)
DSCEngineTest:testRevertsIfMintFails() (gas: 2026539)
DSCEngineTest:testRevertsIfMintedDscBreaksHealthFactor() (gas: 163893)
DSCEngineTest:testRevertsIfRedeemAmountIsZero() (gas: 200261)
DSCEngineTest:testRevertsIfTokenLengthDoesntMatchPriceFeeds() (gas: 181848)
DSCEngineTest:testRevertsIfTransferFails() (gas: 1946641)
DSCEngineTest:testRevertsIfTransferFromFails() (gas: 1967717)
DSCEngineTest:testRevertsWithUnapprovedCollateral() (gas: 708487)
DSCEngineTest:testUserHasNoMoreDebt() (gas: 475044)
DSCEngineTest:testUserStillHasSomeEthAfterLiquidation() (gas: 488630)
DecentralizedStablecoinTest:testCantBurnMoreThanYouHave() (gas: 60278)
DecentralizedStablecoinTest:testCantMintToZeroAddress() (gas: 12893)
DecentralizedStablecoinTest:testMustBurnMoreThanZero() (gas: 60193)
DecentralizedStablecoinTest:testMustMintMoreThanZero() (gas: 12380)
```

 </details>

---

## 🛠️ Tools Used

- Manual audit
- Forge Snapshot
- Bash scripts tailored to analyze forge snapshots.

> 📘 **Notice** ℹ️: I've personally created the bash scripts. Here is a link to the github repo [Forge-Snapshots-Analyzer](https://github.com/CarlosAlegreUr/Forge-Snapshots-Analyzer).

---

## 🎯 Recommendations

Implementing this gas optimization is recommended. This benefits both end-users and the protocol in terms of cost, while ensuring protocol security remains uncompromised as tests are passing.

> 🚧 **Notice** ⚠️: If a significant vulnerability necessitating major code changes arises, this gas optimization technique remains relevant and applicable. Nonetheless some gas analysis as the one used here is always needed to make sure it's actually saving gas.
