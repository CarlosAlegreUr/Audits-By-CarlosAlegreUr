## Vulnerability Details 🔍 && Impact 📈

`offerId` state is meant to be the latest not used ID at `PreMarket.sol`. Yet in `createOffer()` they increase it and then use it, which is inconsistent. Fortunately I could not find any bad consequence as they only mess this up in the struct member `id` and not in the address key that acutally maps by id. So the only bad consequence is inconsistent incorrect state.

As all latter posible operations use the address Id instead of the id memeber of the struct, there is now latter execution problems derived from this.

> ℹ️ **Note for judge** 📘 If there is actually a bad consequence I've missed, I would kindly appriciate a severity increase.

---

## Recommendations 🎯

In `createOffer()`, increase `offerId` latter, not [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L83). But after struct creations, around [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L147).

---