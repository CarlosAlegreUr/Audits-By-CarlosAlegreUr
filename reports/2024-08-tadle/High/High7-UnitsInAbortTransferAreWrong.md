## Vulnerability Details 🔍

In `abortBidTaker()` at `PreMarkets.sol` [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L687), you add to the system's accounting to `_msgSender()` some amount of collateral token. 

This function eventually modifies `userTokenBalanceMap` which is the state tracking what you can and can't withdraw. See [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/TokenManager.sol#L119) the addition and [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/TokenManager.sol#L141) its use in withdrawal.

Yet the units of the variable `transferAmount` [used there](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L691) are not collateral tokens. They are a value with this unit: `points^2/collateral`.

This is because `transferAmount == depositAmount` and `depositAmount` is calculated [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L671), clearly mulitplying: `points * preOfferInfo.points / preOfferInfo.amount`. The amount param of the `Offers` is the amount of collateral deposited so it is in collateral tokens.

## Impact 📈

Wrong accounting of tokens, probably leading to a round down to 0. As points use no decimal precision and collateral is an ERC20 token, usually having 18 decimals. Thus point*point is highly probably < `1e18`. It will only not round down to 0 if `pts^2 >= collateral`. Anyway the wrong amount of tokens will be transferred and probably lost as such small quantities that do not round down to 0 of ERC20 as collaterall are very rare.

Loss of funds as the stock will be marked as `Finished` at the end of the function causing reverts in future trials, see [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L694). See future reverts point [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L657).

---

## Proof Of Concept (PoC) 👨‍💻💻

Detailed execution explanation on why `transferAmount == depositAmount`. A bid taker action can only be linked to an ask offer.

You can easily see `transferAmount == depositAmount` in the `abortBidTaker()` context. As `getDepositAmount()` when passed an ask offer and `false` as parameters, as done in this context, simply returns the function parameter that `depositAmount` occupies, see [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/libraries/OfferLibraries.sol#L41).

Even if this was not the case and `transferAmount == depositAmount * (1 + collateralRate)` (which is the other posible return value of `getDepositAmount()`), the problems of sending wrong amounts that are highly probable to round down to 0 would still be present.

The following test code prooves what I've explained:

<details> <summary> See code 👁️ </summary>

Import `import "forge-std/console.sol";` and paste [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L676) the following logs:

```solidity
    console.log("---------------------------------------------------------");
    console.log("stockInfo.points:    ", stockInfo.points);
    console.log("preOfferInfo.points: ", preOfferInfo.points);
    console.log("preOfferInfo.amount: ", preOfferInfo.amount);
    console.log("depositAmount:       ", depositAmount);
    console.log("---------------------------------------------------------");
```

Then paste this test in the `PreMarkets.t.sol` and run it with `forge test --mt "test_wrongUnits" -vvv`:

```solidity
    function test_wrongUnits() public {
        console.log("-----------------");
        console.log("user -> Alice:  ", address(user));
        console.log("user1 -> Bob:   ", address(user1));
        console.log("-----------------");
        console.log("Alice makes an ask maker offer and bob takes it.");
        console.log("Then all is aborted.");
        console.log("------------------");

        vm.prank(user);
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000,
                1000 * 1e18,
                10_000,
                100,
                OfferType.Ask,
                OfferSettleType.Turbo
            )
        );

        address offerAddr = GenerateAddress.generateOfferAddress(0);
        address stckAddr = GenerateAddress.generateStockAddress(preMarktes.offerId());
        vm.startPrank(user1);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        preMarktes.createTaker(offerAddr, 1000);

        console.log("Calling abortAskOffer and abortBidTaker");
        vm.startPrank(user);
        preMarktes.abortAskOffer(GenerateAddress.generateStockAddress(0), GenerateAddress.generateOfferAddress(0));
        vm.startPrank(user1);
        preMarktes.abortBidTaker(stckAddr, preMarktes.getStockInfo(stckAddr).preOffer);
    }
```

</details>

---

## Recommendations 🎯

Calculate the units properly and get the deisred intended amount. Instead of `point*point/collateral` it should be `point*collateral/point`. 

Im not sure what is the use of `depositAmount`, as of what I understood it's the amount of collateral you payed as a bid taker to get that stock. So if you want a refund in the abort funtion the amount of collateral to account you back would be:

```diff
-        uint256 depositAmount = stockInfo.points.mulDiv(preOfferInfo.points, preOfferInfo.amount, Math.Rounding.Floor);

+        uint256 depositAmount = stockInfo.points.mulDiv(preOfferInfo.amount, preOfferInfo.points, Math.Rounding.Floor);
```

Please note that due to lack of developers answers and not much documentation I'm not sure 100% of the intended use of `depositAmount`. Whatever the use is the units are wrong and that is what the fix should focus on.

---