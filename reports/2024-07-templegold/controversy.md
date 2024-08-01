
# Controversy 😮‍💨

I could not put much effort in this audit. Yet I think the results are quite unfair.

1️⃣ I though of issue `H-01` in [the report](https://codehawks.cyfrin.io/c/2024-07-templegold/results?lt=contest&page=1&sc=reward&sj=reward&t=report) about the cross-chain bridge being incomaptible with some kind of wallets. But I did not submit it because I deemed it `Informational` and design choice, yet the judge decided to label it as `High`.

2️⃣ The issue I submitted was labeled as `Invalid: Informational`. I completely disagree it is at least a Low. The issue is [this one](./High2-HardForkWillDeemTempleGoldUseless.md), about a hard-fork changing the `chainId`. The invalidation reason was that the main chain will keep the `chainId`, this is not the point of the issue. If the protocol doesnt like the new mechanics of the new main chain and wants to keep operating in the old `Arbitrum` they would not be able to do so. Protocol says they want to operate in `Arbitrum` but if it hard-forks there will be 2 `Arbitrum` and the protocol can't decide which to chose, possible being locked in the one they actually dislike. Sadly, I did not have time to escalate this issue. 
