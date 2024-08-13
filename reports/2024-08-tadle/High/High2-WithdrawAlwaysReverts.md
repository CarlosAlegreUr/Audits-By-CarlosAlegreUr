## **Vulnerability Details 🔍 && Impact 📈**

Users can deposit value in collateral tokens on the code but can't withdraw it. This is because `TokenManager.sol` does not call the `approve()` function in `CapitalPool` before using `_safe_transfer_from()` inside the `withdraw()` function.

---

## **Proof Of Concept (PoC)** 👨‍💻💻

Paste this test in the `PreMarkets.t.sol` file, import `import "forge-std/console.sol";`, and run `forge test --mt "test_withdraw" -vvv`:

<details> <summary> See test code 👁️ </summary>

```solidiy
   function test_withdraw() public {
        console.log("-----------------");
        console.log("user -> Alice:  ", address(user));
        console.log("user1 -> Bob:   ", address(user1));
        console.log("-----------------");

        console.log("Alice makes an ask maker offer and bob takes it.");
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
        console.log("1e18 reference:                        ", 1e18);
        console.log("Alice should send 1000$ as collateral: ", b_cache - b_after);

        address offerAddr = GenerateAddress.generateOfferAddress(0);
        console.log("Bob takes the whole offer for 1000$");

        vm.startPrank(user1);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        preMarktes.createTaker(offerAddr, 1000);

        console.log("See Alice balance on tokenManager: ");
        uint256 alceBalance =
            tokenManager.userTokenBalanceMap(address(user), address(mockUSDCToken), TokenBalanceType.SalesRevenue);
        console.log("Alice balance Sales Revenue:           ", alceBalance);
        console.log("As you will see in foundry logs, it reverts due to lack of allowance.");
        console.log("If adding the line in Recomendations section and run test again all works.");
        vm.startPrank(user);
        tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.SalesRevenue);
    }
```

 </details>

---

## **Recommendations 🎯**

Add this line of code inside the `withdraw()` function, [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/TokenManager.sol#L173) :

```solidity
+ ICapitalPool(capitalPoolAddr).approve(_tokenAddress);
```

---
