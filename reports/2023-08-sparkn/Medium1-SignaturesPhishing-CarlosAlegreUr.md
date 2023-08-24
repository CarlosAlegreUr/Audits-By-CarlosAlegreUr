## **Summary 📌**

The choice to utilize **_delegated signatures_** through **EIP712** for distribution delegation presents a **vulnerability**. Not from a coding oversight, but due to **human mistakes**, especially from the target audience of this feature - **crypto newcomers** as indicated by the developers.

---

## **Vulnerability Details 🔍**

As explained in the [developer's walkthrough video (24:10)](https://www.youtube.com/watch?v=_VqXB1t9Evo&t=1450s), **meta transactions** paired with **signatures** aim to make the protocol more approachable for those unfamiliar with crypto.

However, there's a plausible scenario where these new users may **not fully recognize** the weight of their signature or how the EIP712 message they wanna sign should look like. This **blind spot** could pave the way for malicious actors to employ conventional deceptive tactics, such as convincing users to sign malicious actions, using **real-world deceit**, **mimicking emails**, or leveraging **`URL hijacking`** techniques.

> 📘 **Note** ℹ️: `URL hijacking` involves domain names designed to trick users due to their resemblance to popular websites. A classic example is "faceb`o`k.com" versus the genuine "faceb`oo`k.com". These deceptive domains can host **phishing sites** that manipulate user's data with malicious intents.

If a threat actor manages to **make a user create a malicious signature**, be it from web-based threats, deceit, or other methodologies, they gain the power to **redirect prize distributions**, potentially leading to **unauthorized fund withdrawals**.

---

## **Impact 📈**

This presents a **_risky scenario_**, potentially culminating in the **misappropriation** of organizer or sponsor funds. This would lead to **damaging the protocol's reputation** from the users perspective even though is not a technical fault inside the contracts' code.

As the loss of funds happens not in the code but when humans use the code wrongly, I decided to label the finding as `Medium` risk.

---

## **Tools Used 🛠️**

- **Manual audit**.
- Insights from the [Developers walkthrough](https://www.youtube.com/watch?v=_VqXB1t9Evo&t=8s).

---

## **Recommendations 🎯**

1️⃣ **_Prioritize user education_**. Ensure that organizers **understand the importance** of their signature. And warn them about things like: SPARKN will **never** message you asking for signing messages apart from within the official website: _`exampleSparkin.com`_. And a sigining distribution message should look like: _`some_real_example`_.
