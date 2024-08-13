## Summary 📌

It is very complex to determine if a turbo ask offer has still chances of being actually settled. Due to the complexity of the process I consider this a great risk for honney pots that trick users to buy offers that no-more have a real penalty for not settling them to the ask maker. I consider this of great likelihood for people being scammed and thus sent it as a Medium severity issue.

This problem at its core comes from the possibility of creating turbo offers in which: `eachTradeTaxPercentage >= collRatioPercetage + gasCosts`. `gasCosts` in most chains and cases can be considered negligable.

If this condition is given, the offer made is basically a honeypot, as the seller has no incentive to settle the tokens, as they receive more money from selling its points than the collateral that backs them up in case of defaulting. Yet this condition is an assurance of scam, the condition not being met is not an assurance of no-scam. And the way to really make sure of it is complex and explained latter.

---

## Vulnerability Details 🔍 

### 1️⃣ Understanding the problem

The system has an inherent unavoidable risk which is: after token lauch or TGE (token generation event), if the value of those tokens is higher than the collateral, the seller will have no incentive to settle the tokens, as they can just sell them in a DEX for more profit. This is acceptable and parties participating know it.

But makers can create turbo offers that are straight-up scams yet difficult to detect, as they assure that its settlement is never worth it for the seller. 

This is because they receive more money from selling its points than the collateral that backs them up in case of defaulting. This can lead to a honeypot situation where users are attracted to offers that were never meant to be settled. As I will explain further, detecting this offers is not trivial and I consider this honeypots a realistic trap a normal user could fall into.

Simplifying, for the seller to make more profit in the pre-market that the value of its collateral, thus having no incentive to settle, this has to be met: `collValue < tknsValue + profitFromPoints + profitFromTax`.

A good example of offer settings that leads to this is in the very same docs of the protocol, [here](https://tadle.gitbook.io/tadle/how-tadle-works/mechanics-of-tadle/turbo-mode). In all that process (Section: `How Turbo Mode works -> Sell Offer on Turbo Mode`) Alice has 0 incentive to settle tokens as she has earned as much as its collateral value by selling points. AliceColl == 1000$ , and her selling earns are worth 1000 $ . Now when she gets the TGE tokens which will have a value greater than 0$ in DEXEs, she will own `sellingRevenue + tknValue` which is clearly greater than Alice's collateral. Notice that if the market was centralized or there was KYC, losing reputation might be a risk for Alice, but this is not the case in Tadle so there is no risk.

### 2️⃣ Why are these honeypots hard to detect and therefore a risk?

Imagine you are a user, and you see an offer which a lot of people are trading at a good price (most of them might even be fake addresse owned by the same firt asker, but you can't know this as addresses are anonymous).

To detect if this offer is a honeypot you must:

- 1️⃣ Check for this condition first: `eachTradeTaxPercentage > collRatioPercetage`. If this is given, the offer is starightup a scam. Yet if its not given it does not mean it is not a scam.

<details> <summary> Understanding this inequality 🤔 </summary> 

***Understanding this equality***: for every point you sell at a price there is a greater amount of collateral backing them up. But in each sell an extra amount of tax is added (see [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L829)), so if you are overcollateralized by 10% and the tax is 11%. For every 10% extra collateral that you deposited per token, you are getting in return 11% more for each token sold. A 1% profit. Notice that `platformFee` is payed here by the taker and thus won't affect your profit as ask maker.

Generally: to sell X points at Y price you need to be overcollateralized by a certain percentage. So you put the offer and the amount of value the points are worth which is `X*Y = pts * $/pts = $`, but because is overcollateralized you have to transfer `X*Y*(1+collRate)`. Then when you sell them to a taker they are charged that same price for the same amount of points plus the trade tax => `X * Y + (X*Y*tradeTax)`.

Then for every point being sold your revenue (`$/pts`) is `(X * Y + (X*Y*tradeTax)) / X`. And, for you to create that offer you put an overcollateralized position which its valued at: `X*Y*(1+collRate)`. All that means that creating the offer cost you: `X*Y*(1+collRate) / X` $/pts. So if we equal the cost with the benefits to see where we break even we get:

`profitPerTokenSold == costPerTokenDeposited`

`(X * Y + (X*Y*tradeTax)) / X == X*Y*(1+collRate) / X` (simplify this and...)

`tradeTax == collRate` => we break even when this condition is meat.

We could have solved this as `>`, no negative numbers are involved so there is no sign flip and we get that the profit per token sold is greater that the cost of selling them when: `tradeTax > collRate`

</details> 

- 2️⃣ The original maker could have made more profit by selling points even when `tradeTax < collRate`. The intial maker of a turbo ask offer will receive the tax income from any subsequent trades. If the trades number is large enough, even with a small competitive trade tax percentage the original maker will have earned enough so as to not to not care about settling the offer.

Honeypot scammers can play with this settings, ghost addresses and competitive taxes and re-sales to make a profit. With little to no cost as they can always cancel offers and mititage risks.

> ℹ️ **Note** 📘 Notice that in `Protected` mode this selling scams are actually easy to detect, there the `tradeTax < collRate` check is sufficient.

## Impact 📈

It is pretty complex to determine if a turbo ask offer has still chances of being actually settled. Average user won't likely take all the time to analyze offers and will just see prices and trading activity to decide the maker to buy to. Average user will see and think something like: this price for this future token I want seems fine and people are trading so it must be cool seller. And will purchase the scam.

---

## Recommendations 🎯

- 1️⃣ Do not allow creation of turbo ask offers which: `tradeTax >= collRate`. As they are useless due to not having settling incentives and can easily be used for scams. (same in `Protected` mode)

- 2️⃣ Track the aggregated profit a turbo seller has made form re-sales and when that profit is within a certain threshold of exceding collateral value revert the txs that try to resell the points.

All this will ensure that turbo mode still generates more dynamic liquidity without leaving space for likely scams.

---
