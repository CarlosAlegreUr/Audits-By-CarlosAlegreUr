## Vulnerability Details

If an already existing **stake.link** v1 `OperatorVault` desires to use **stake.link** for v2 it will revert on its upgrade, whereas new operators that join **stake.link** will create their new contract successfully. Furthermore `OperatorVCS` contract upgrade will also fail.

The reason is because of the `reinitializer(3)` modifier in the `initialize()` functions.

## Proof Of Concept

If you go the official `OperatorVCS` for v1 in Etherscan [here](https://etherscan.io/address/0x4852e48215A4785eE99B640CACED5378Cc39D2A4#readProxyContract) you will see that the proxy `_initialized` variable has the value 3.

If you use `getVaults()` and go the first vault address as example (see [here](https://etherscan.io/address/0x8d87CBD8C3632b7ef117A15F8100943a23b7D03b#readProxyContract)) you will also see that `Vaults` have `_initialized == 3`.

To read the private state from Etherscan you can use this browser extension [BlockSec Meta Suits](https://blocksec.com/metasuites).

Now this is the `reinitializer()` code used by the system:

```solidity
modifier reinitializer(uint8 version) {
    // 👁️🔴⏬ `_initialized < version`, this condition is not met as `3 < 3 == false` 
@>    require(!_initializing && _initialized < version, "Initializable: contract is already initialized");
    // code...
}
```

## Impact

Previous operators that want to migrate to v2 will not be able to do so. And the upgrade for the `OperatorVCS` will not be possible either.

## Recommendations

Reinitializer should be `reinitializer(4)` in both `OperatorVaults` and `OperatorVCS`.
