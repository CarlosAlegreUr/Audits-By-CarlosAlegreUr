## Relevant Context

There are `deposit()` and `depositAndCall()` functions for `ZRC20`s in `GatewayZEVM`. Yet for Zeta there is only `depositAndCall()` function.

## Description

If someone initiates a cross-chain tx from spoke chain to Zeta chain to send Zeta to an EOA, eventually `depositAndCall()` gets executed and tries to call:

```solidity
UniversalContract(target).onCrossChainCall(context, zetaToken, amount, message);
```

Being `target` an EOA, the tx will revert. Thus now the `Observer` will catch this and call `depositAndRevert()`. Yet if the EOA is set ass `revertAddress` on `RevertOptions` it will revert again in:

```solidity
UniversalContract(target).onRevert(revertContext);
```

## Impact Explanation

Directly sending Zeta to an EOA from spoke to Zeta is meant to work seamlessly but it doesn't, it will just always revert.

## Proof of Concept

Paste this code in [Remix](https://remix.ethereum.org/) and see that in EVM and this Solidity version these types of calls to EOAs revert.

Here is a famous EOA you can use as input: [0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045](https://etherscan.io/address/0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

interface UniversalContract {
    function onCrossChainCall() external payable;
}

contract CrossChainCallTest {
    function unsafeCrossChainCall(address target) external payable {
        UniversalContract(target).onCrossChainCall() ;    
    }
}
```

## Recommendation

Add the missing `deposit()` function for Zetta tokens in `GatewayZEVM` to allow direct sending to EOA.

Or: Add a check in `depositAndCall()` to do not call `onCrossChainCall()` if `target` is an EOA, like so: `require(target.code.length > 0, "Target must be a deployed contract");`.
