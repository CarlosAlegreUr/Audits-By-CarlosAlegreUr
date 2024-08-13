## Summary 📌

The code uses unsafely the `.mulDiv()` function, this function ultimately does: `number*numerator/denominator`. There are realistic scenarios where: `number*numerator < denominator` and the division truncates to 0.

This can ultimately be used to avoid taxes with some tokens or avoid platform fees too. Referral codes can potentially get affected by this too

---

## Vulnerability Details 🔍 && Impact 📈

Trade tax is calculated like so:

`uint256 tradeTax = depositAmount.mulDiv(makerInfo.eachTradeTax, Constants.EACH_TRADE_TAX_DECIMAL_SCALER);`

Which in math is: `depositAmount * makerInfo.eachTradeTax / Constants.EACH_TRADE_TAX_DECIMAL_SCALER`

`makerInfo.eachTradeTax <= Constants.EACH_TRADE_TAX_DECIMAL_SCALER`. Assuming healthy competitive market taxes will go down to offer better trades so its safe to assume that most of the time `makerInfo.eachTradeTax < Constants.EACH_TRADE_TAX_DECIMAL_SCALER`. Thus the division `makerInfo.eachTradeTax / Constants.EACH_TRADE_TAX_DECIMAL_SCALER` in solidity will round to 0 if `depositAmount` multiplied by `makerInfo.eachTradeTax` is not strong enough to make it `> Constants.EACH_TRADE_TAX_DECIMAL_SCALER` and get the right calculation.

- Numerical Example 👁️:  `EACH_TRADE_TAX_DECIMAL_SCALER==10_000`, see [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/libraries/Constants.sol#L14). And lets say a competitive tax of `1% == makerInfo.eachTradeTax == 100`. If `depositAmount < (10000 / 100)` it will round down to 0.

You might notice that for ERC20s with 18 decimals an amount of < 100 is ridiculous and gas cost would be higher than tax savings. But for tokens of <= 2 decimals this is a real problem. As these tokens if used, small purchases from modest clients won't pay taxes. If you payed lets say `3$ == depositAmount == 300`, then `300*100/10000 == 3000/10000 == 3$`. But in solidity `3000/10000` truncates to 0.

A 2 decimal real token example GUSD (Gemini dollar), [see etherscan](https://etherscan.io/address/0x056fd409e1d7a124bd7017459dfea2f387b6d5cd#readContract).

This problem will probably occure in other parts of the code that use `.mulDiv()` to calculate token amounts mulitiplied by some percentage ratio based on the scalars. Like in calculating `platformFee` [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L219). Notice that the scaler here is `1_000_000` so tokens with more than 2 decimals can be problematic too. Like famous and super-used USDC with 6 decimals.

---

## Proof Of Concept (PoC) 👨‍💻💻

Paste this test in the `PreMarkets.t.sol` file, import `import "forge-std/console.sol";`, and run `forge test --mt "test_skipTax" -vv`. You should also ovrride the `decimals()` function in the mock contract used by tests, [this one](https://github.com/Cyfrin/2024-08-tadle/blob/main/test/mocks/MockERC20Token.sol). Like so:

```solidity
  function decimals() public pure override returns (uint8) {
        return 2;
    }
```

<details> <summary> See test code 👁️ </summary>

```solidity
    function test_skipTax() public {
        console.log("-----------------");
        console.log("user -> Alice:  ", address(user));
        console.log("user1 -> Bob:   ", address(user1));
        console.log("-----------------");

        console.log("Alice makes an ask maker offer and bob takes it without paying the tax.");
        console.log(
            "Alice sets a competitive low tax, 1%. Notice that in a competitive enviroment taxes tend to be as low as possible."
        );
        console.log("The lower the tax the greater the amount you can buy skipping it.");

        vm.prank(user);
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000,
                500 * 1e2,
                10_000,
                100,
                OfferType.Ask,
                OfferSettleType.Turbo
            )
        );
        address offerAddr = GenerateAddress.generateOfferAddress(0);
        console.log("Bob takes 1 pts for 0.5$, 1% tax is 5 cents = 5");

        vm.startPrank(user1);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        preMarktes.createTaker(offerAddr, 1);

        console.log("See Alice balance on tokenManager: ");
        uint256 alceBalance =
            tokenManager.userTokenBalanceMap(address(user), address(mockUSDCToken), TokenBalanceType.TaxIncome);
        console.log("Alice balance Tax Revenue:           ", alceBalance);
        console.log("Alice should have 5 cents as revenue but round down to 0.");
        console.log("Only cost to avoid tax is gas cost, in layer 2 evm compatible where gas is very cheap < tax");
        console.log("You can just multicall and avoid tax.");
    }
```

</details>

---

## Tools Used 🛠️

- Manual review.

---

## Recommendations 🎯

There are multiple approaches to mitigate or even elimiate this issue:

- Reducing the constant scaler to something smaller than `10_000` in Layer2s or blockchains where gas costs are low to make it harder for this skip to be profitable.

- Another approach and the one I recommend the most would be, when an operation carries taxes, detecting if the multiplier `amount * numerator <= denominator` and if so revert saying, minimal amount for avoinding tax evasion is not reached.

> ℹ️ **Note** 📘 There is probably better ways of fixing this, I do not have enough time to get deeper into it. I encourage the devs to do so.

---