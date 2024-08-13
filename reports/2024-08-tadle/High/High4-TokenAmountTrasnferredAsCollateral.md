
## **Vulnerability Details 🔍 && Impact 📈**

At `closeBidTaker()` at `DeliveryPlace.sol`, `tokenManager.addTokenBalance()` is called like so:

```solidity
 tokenManager.addTokenBalance(
    TokenBalanceType.PointToken,
    _msgSender(),
    makerInfo.tokenAddress, // 🔴⚠️ COLLATERAL TOKEN ADDRESS!
    pointTokenAmount // 🔴⚠️ TGE TOKEN AMOUNT! decimals can differ for example
);
```

See line [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/DeliveryPlace.sol#L198).

It clearly mixes an amount of one token with the address of other token.

The impact is that a TGE token amount is accounted for as if it where collateral. If this tokens differ in decimals, lets say token has 2 more decimals than collateral. Then `_msgSender()` would be gainning access to withdraw a really big amount of collateral that clearly does not belong to him. And viceversa if collateral has more decimals than TGE token the user would be receiving way less collateral than he should. Also after closing, offers and stocks are marked as `Finished` or `Settled`, leaving any wrong amount of tokens incorrectly accounted for stuck in the contract.

This also happens in `settleAskTaker()`. See [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/DeliveryPlace.sol#L387).

---

## **Recommendations 🎯**

Use the `marketPlaceInfo.tokenAddress` which is the actual token address of the TGE token. And it is correcly used in other functions of the protocol like [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/DeliveryPlace.sol#L379). This is the actual token backed by points address that are delivered on settlement. The error is visible here as after transferring tokens with the `tillIn()` the protocol acounts that amount with `addTokenBalance()`, yet immediately after there they wrongly use the collateral address, [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/DeliveryPlace.sol#L387). 

---