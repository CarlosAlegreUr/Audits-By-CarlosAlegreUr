# Project ℹ️

🔗 [2024-03-woofi-swap](https://github.com/sherlock-audit/2024-03-woofi-swap)

🔗 Competition details on **sherlok**: [click here](https://audits.sherlock.xyz/contests/277)

According to the developers:

---

_`WOOFi is a cross-chain DEX where anyone can swap, stake, and earn crypto and trade perpetual futures across every major blockchain.`_

---

# Rewards Earned 💸🧠

- Experience and knowledge. 😄
- 3992.17 $ 💸

# Lessons Learned 🧑‍💻

- A different algorithm for MM rather than AMM, PMM and sPMM whih is specific for the project.
- How Routers and DEXEs actually deeply work.

# _The reports_ 📝

Check the findings' reports I submitted:

#### High 🔴

- [🔗 High1-SwapsDirectlyOnWooPPCanBeFrontRun](./High/High1-SwapsDirectlyOnWooPPCanBeFrontRun.md) (`Invalid`: intended behaviour)

- [🔗 High2-UsersSkipProtocolFeesWithEvilToken](./High/High2-UsersSkipProtocolFeesWithEvilToken.md) (`Invalid`: accepted behaviour)

- [🔗 High3-UserSkipProtocolFeesWithMinAmount](./High/High3-UserSkipProtocolFeesWithMinAmount.md) (`Invalid`: accepted behaviour)

#### Medium 🟡

- [🔗 Medium4-UsersPayExternalFeesWhenTheyShouldnt](./Medium/Medium4-UsersPayExternalFeesWhenTheyShouldnt.md)

- [🔗 Medium5-CrossChainWETHSwapFeesChargedUnnecessarily](./Medium/Medium5-CrossChainWETHSwapFeesChargedUnnecessarily.md) (`Escalating`)

- [🔗 Medium6-UserReceivesLessThanMintToLimit](./Medium/Medium6-UserReceivesLessThanMintToLimit.md)

#### Low 🔵

- [🔗 Medium1-ImmutableAddressesOfProxies](./Medium/Medium1-ImmutableAddressesOfProxies.md) (`Non-rewarded`: downgraded to **low**)

- [🔗 Medium2-ChainlinkDataRetrievedUnsafely](./Medium/Medium2-ChainlinkDataRetrievedUnsafely.md) (`Non-rewarded`: downgraded to **low**)
 
- [🔗 Medium3-RefundThroughCrossChainWidgetIsWrong](./Medium/Medium3-RefundThroughCrossChainWidgetIsWrong.md) (`Non-rewarded`: downgraded to **low**)