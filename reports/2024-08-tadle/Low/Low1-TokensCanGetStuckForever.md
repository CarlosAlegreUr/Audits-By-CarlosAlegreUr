## **Vulnerability Details 🔍**

Imagine a scenario where the `TGE` has already happened but no-one claims their tokens or collateral in case of default.

Assuming the protocol accounting is not broken, only the `Stock.authority` or `Offer.authority` can withdraw the value from the system.

Also the `owner()`, if he holds `TGE` tokens could eventually settle and retreive the collateral, yet this depends on the owner having the token, which is a risk.

## Impact 📈

If these addresses' private key is lost or users forget to claim their share and the owner does not hold the token being distributed, funds will be there forever. Chances of this occuring is _Low_ but possible.

---

## **Recommendations 🎯**

Add an expiration date on each `Offer` and `Stock` to avoid the funds being stuck forever.
After the expiration date anyone can claim those funds or maybe just send them to the dev team wallet.

---
