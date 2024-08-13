## Vulnerability Details 🔍 && Impact 📈

Upgrades in `TadleFactory` can damage users due to deploying contracts unpaused by default. This makes the new implementation immediately active and can affect users that are currently executing transactions.

If a user sends a tx, lets say with a taker order, and now **Tadle** just happens to upgrade to a new verion where takers pay some extra fee due to some new cool feature and the upgrade tx happens before the taker tx, the taker will pay unexpected fees.

The upgrade will take place and latter the taker would take an offer and pay the fee or whatever the new feature is will be applied to him unexpectedly in the new version. The fee is just an example, it could be any other feature that could affect the user.

This is why new upgrades should be deployed paused by default and unpause them after a few blocks have passed to assure no-one is executing tx that they expected to be executed in older versions.

This issue can me mitigated by pausing the old implementation before deploying the new one all if desired in a multicall. But there is still this risk of tx ordering in the block: ***1st -> mutlicall -> 2nd -> user taker tx***.

Furthermore, upgrades being immediately availabe are also risky due to re-orgs. Imagine a user sees that the upgrade has happened and sends tx to operate on that yet a re-org happens and the tx is executed in the old version. This could lead to unexpected results.

---

## Proof Of Concept (PoC) 👨‍💻💻

In `TadleFatory`, the mapping that is read from other system contracts gets updated immediately in the `deployUpgradeableProxy()`. See [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/factory/TadleFactory.sol#L68).

Imagine `TokenManager` gets updated with more fees or whatever other feature. The taker sent the `createTaker()` tx to `PreMarkets`, then [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L226) the upgraded address would be read from the updated mapping and the tx would be executed in the unexpected version of `TokenManager`.

See [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/libraries/RelatedContractLibraries.sol#L57) that the function from the libray called in the previous link actually reads directly from `TadleFactory`'s mapping.

---

## **Tools Used 🛠️**

- Manual review.

---

## **Recommendations 🎯**

Somehow give users time to react on upgrades. An example would be to deploy upgrades paused by default and unpause them after a few blocks have passed (enough to delete re-org risks). During this time add a revert message like: `Contracts Upgrading Prevetion Revert`. To inform the user that his tx might have differnt results than expected if re-sent and please read the new features before operating.

---