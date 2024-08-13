## Vulnerability Details 🔍 && Impact 📈

In a trade where the ask side is the taker side, latter when settling only the bidder is authorized to do so. This makes no sense as bidder has no TGE tokens it is the asker side that has them.

See at `settleAskTaker()` [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/DeliveryPlace.sol#L361) that the `_msgSender()` has to be the offer authority, yet bidder is the maker side creating the offer and thus its authority.

Latter tokens are also taken from `_msgSender()` which is the bidder side, see [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/DeliveryPlace.sol#L377).

The impact is that this won't be settled as the expected caller for settling this as of code is written now is the bidder and the bidder is the receiver of TGE tokens, not the settler.

---

## **Recommendations 🎯**

Change the check on the if statement to:

```diff olidity
if (status == MarketPlaceStatus.AskSettling) {
-            if (_msgSender() != offerInfo.authority) {
+            if (_msgSender() != stockInfo.authority) {
                revert Errors.Unauthorized();
            }
    }
```

---
