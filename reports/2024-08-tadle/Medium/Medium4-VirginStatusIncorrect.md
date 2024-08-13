## Summary 📌

There are several issues and inconsistenies that arise from the way the protocol uses the `OfferStatus` enum.

Due to lack of answers from devs, incomplete documentation that contradicts the code sometimes, and even the same code that seems to mean different things. It is hard to determine if some of the issues are real or not.

There are 3 [OfferStatus](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/storage/OfferStatus.sol#L15) not used at all: `Filled`, `Settling` and `Ongoing`. And the `Virgin` status is wrongly applied according to its definition.

The inorrect use of `Virgin` is for sure an issue, and another issue can be derived from the lack of use of the other 3 states. Yet due to the ambiguity of the codebase, it is hard to determine if the arising issues are intended or not. I will report these issues as a collective one in another submission.

---

## Vulnerability Details 🔍 && Impact 📈

### `Virgin` issue

A real code error that for sure can be submited is that the `Virgin` state usage is inconsistent. `Virgin` offers are meant to be the ones which no taker has interacted with, yet we see that when a taker takes an offer this one is still being marked as `Vrigin`. Which is incorrect and should cause problems down the line. Yet the code seems to acknowledge this and modify the functions in such a way that this does not cause any issues:

- At `settleAskMaker()` for example only `Virgin` or `Cancelled` offers can be setteld, but this does not make sense according to the definition as virgin offers are ones that sold nothing. Settling to no-one makes no sense still the function if you sold 0 tokens and you settle 0 tokens gives your collateral back which is a valid interaction and an expeted posible action to make when you don't get any taker to your maker offer. So a function that by definition should not accept `Virgin` offers, it does and it even does it acknowledging its definition and executing a valid action on them.

Inside this very same function, [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/DeliveryPlace.sol#L278) you see that if the offer is `Vrigin` the refund is equal to the total amount of collateral, which is correct if the offer has no takers. And if not `Virgin` only the sold tokens are taken into account. This is a clear sign that the code is aware that `Virgin` offers have no takers and if they do have takers and have sold some part of tokens, then they are not `Virgin`. Yet in `createTaker()`, when offers are answered by a taker, its staus is never changed and remains `Virgin`, which is incorrect. See the function [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L39), you will never see such a thing as `offer.offerStatus = SomeOtherNewStatus`.

- [Here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L586) if offer is `Virgin` it takes the whole amount because it assumes nothing is sold from it. If not `Virgin` takes a proportional amount of the offer to the points sold. Again another part of the code that acknowledges the `Virgin` status is meant to be the one that has not sold any points.

Yet along the codebase when a taker takes action it never changes the `Virgin` state of an offer.

---

## Recommendations 🎯

Use the other 3 states defined as they are supposed to work and also refactor the code making `Virgin` atually behave as its definition says.

---