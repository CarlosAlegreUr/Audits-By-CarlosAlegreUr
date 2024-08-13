## Vulnerability Details 🔍 && Impact 📈

At `TadleFatory.sol` the address `guardian` is the one in charge of deploying contracts and/or updating them.
Yet this address is set in the constructor and latter there is no way of changing it.

Should this address be compromised, there would be no way for a new legit guardian to at least try gain control of the system.

A new guardian could make `relatedContracts` mapping point to whatever he wants, and this mapping is in key parts across the system like the `CapitalPool` or in the `onlyRelatedContracts` modifier at `TokenManager`.

See [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/factory/TadleFactory.sol#L21) the variable. At first glance as the contract only has 70 lines you can see there is not way of changing the value. None inherited contracts are related to this value either.

***

## Recommendations 🎯

Create a 2tx process to change the guardian address, or at least add a way to change it.
Something similar to `Ownable2Step` by OpenZeppeling for example.

***
