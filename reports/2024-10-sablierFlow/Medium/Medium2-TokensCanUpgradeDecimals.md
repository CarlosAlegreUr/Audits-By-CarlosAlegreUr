## Vulnerability Details

As per the [contest docs](https://codehawks.cyfrin.io/c/2024-10-sablier) ***Scope/Compatibility***, ERC20 tokens which are upgradeable are supported.

An upgradeable token can upgrade/change its decimals.

## Impact

If this happens all calculations in the system involving the cached `tokenDecimals` at `_streams[streamId]` storage will be incorrect. As this is only set once at creation time [here](https://github.com/Cyfrin/2024-10-sablier/blob/main/src/SablierFlow.sol#L600). There are a lot of functions that read from this state, to name one check this one on the `_withdraw()` function, see [here](https://github.com/Cyfrin/2024-10-sablier/blob/main/src/SablierFlow.sol#L820).

If decimals increase, cached version will be calculating greater amounts than it should. And vice versa, if decimals decrease, old cache version will be calculating smaller amounts than it should. Leading to transferring more or less tokens that the **Flow** should, leading to a loss of funds to the receiver or the sender which would send or receive more or less tokens than expected.

Furthermore, a token cached with 18 decimals can also increase its decimals to > 18, leading to incorrect calculations in `Helpers::descaleAmount()` and `Helpers::scaleAmount()` functions. As `(18 - decimals)` would underflow to very big numbers due to being inside an `unchecked` block returning very big numbers or rounding down to 0 in `scaleAmount()` and `descaleAmount()` respectively. See these functions [here](https://github.com/Cyfrin/2024-10-sablier/blob/main/src/libraries/Helpers.sol#L59).

A simple example of a problematic use of the miscalculation in `scaleAmount()` can be seen in `_void()` [here](https://github.com/Cyfrin/2024-10-sablier/blob/main/src/SablierFlow.sol#L748). Where even if the `balance` is not enough to cover debt, due to over-scaling it will allow the receiver to transfer out more debt than it should. Stealing from other users. The system indeed warns that does not support tokens with decimals > 18 and this is an example of why.

Summing up, for the different reasons explained, not checking for changes in decimals is a critical risk which can happen due to expecting upgradeable ERC20 tokens.

+ Likelihood: Low
+ Impact: High
+ Severity: Medium

## Recommendations

2 ideas for solutions:

1️⃣ Fetch decimals every time it is used and update state accounted in token decimals if changed.

2️⃣ Allow users to update it in case it changed. Something similar to:

```solidity
// 🔴 This is a simplified version!: All values representing token amounts in token decimals
// 🔴 related to the stream must be updated
function updateTokenDecimals(uint256 streamId) public {
    IERC20 token = IERC20(_streams[streamId].token);
    if(token.decimals() != _streams[streamId].tokenDecimals) {
        _streams[streamId].tokenDecimals = token.decimals();
    }
}
```
