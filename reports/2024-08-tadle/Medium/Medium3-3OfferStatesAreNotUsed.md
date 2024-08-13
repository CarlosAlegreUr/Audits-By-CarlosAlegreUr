## Summary 📌

There are several issues and inconsistenies that arise from the way the protocol uses the `OfferStatus` enum.

Due to lack of answers from devs, incomplete documentation that contradicts the code sometimes, and even the same code that seems to mean different things. It is hard to determine if some of the issues are real or not.

There are 3 [OfferStatus](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/storage/OfferStatus.sol#L15) not used at all: `Filled`, `Settling` and `Ongoing`. And the `Virgin` status is wrongly applied according to its definition.

The inorrect use of `Virgin` is for sure an issue and I've submitted it separately from this one. This one showcases issuees that can be derived from the lack of use of the other 3 states. Yet due to the ambiguity of the codebase, it is hard to determine if all arising issues are intended or not. Just in case, here I report them.

---

## Vulnerability Details 🔍 && Impact 📈

`Filled`, `Settling` and `Ongoing` offer statuses are never used. Due to lack of complete documentantion respecting this and the devs not answering questions probably due to an overload of them, I can't deide on time if this has been intentional or an oversight. Thus the validity of all issues related to the lack of use of this states is not clear. I do think the great ambiguity on the topic should deem as 1 valid collective issue anyway. This part of the protocol about `OfferState` transitions is unauditable because for an audit to be taken you should clearly know what the code is meant to mean and do.

Here is a clear example of the confusion and contradictory information in the codebase:

- Settling in 2-txs is imposible as the `Settling` status is not used. If you partially settle tokens the offer will be marked as `Settled` and revert in future transactions ([here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/DeliveryPlace.sol#L243)) that pretend to fully complete settling the offer. The code indicates this is not meant to be allowed but the existance of the `Settling` status indicates it was probably meant to be allowed. This is what I meant by contradictory information due to code actions, meanings and poor documentation which makes the code unauditable as you can't know if this is a valid finding or just intended behavior. Anyway here I report it.

Other errors derived from the lack of usage of these states are the obvious ones:

- `Filled` should be used when all points are sold and it is not (this was said by the dev in a code walktrhough, exactly [here](https://www.youtube.com/watch?v=JLaqo4cBB40&t=1906s)) being used.

- `Ongoing` seems to be intended to be used when only a part of the points are sold, but it is not being used.

---

## Recommendations 🎯

Clearly define what the code is meant to do and mean with every `OfferStatus` defined and then refactor and reaudit the code.

> ℹ️ **Note** 📘 If the inorrect usage of the 3 states is deemed as 3 different issues I would appreciate this issue being deemed as 3 valid ones. Sorry if this is not the correct way of reporting this, this contest is pretty confusing.

---