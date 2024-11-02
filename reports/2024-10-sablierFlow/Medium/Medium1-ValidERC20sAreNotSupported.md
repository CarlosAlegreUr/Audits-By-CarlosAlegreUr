## Vulnerability Details

As per the [ERC20 standard](https://eips.ethereum.org/EIPS/eip-20#decimals), the `decimals()` function is **OPTIONAL**.

Thus, some valid ERC20 tokens may not have the `decimals()` function implemented. Thus reverting on the `_create()` function [here](https://github.com/Cyfrin/2024-10-sablier/blob/main/src/SablierFlow.sol#L579):

```solidity
uint8 tokenDecimals = IERC20Metadata(address(token)).decimals();
```

## Impact

Expected supported ERC20 tokens, as per the [contest docs](https://codehawks.cyfrin.io/c/2024-10-sablier) ***Scope/Compatibility*** section and ERC20 specification are not supported.

## Recommendations

Add to the list of supported tokens those that have the `decimals()` function implemented.
