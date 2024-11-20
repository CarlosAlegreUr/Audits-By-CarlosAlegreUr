## Description

This is the `calculatePortion()` at `BipsLibrary.sol` on **v4-core**:

```solidity
/// @param amount The total amount to calculate a percentage of
/// @param bips The percentage to calculate, in bips
function calculatePortion(uint256 amount, uint256 bips) internal pure returns (uint256) {
    if (bips > BPS_DENOMINATOR) revert InvalidBips();
@>    // ⏬👁️🟢 `amount * bips` can be < BPS_DENOMIATOR and round down to 0
    return (amount * bips) / BPS_DENOMINATOR; 
}
```

## Impact Explanation

This function is called by the `UniversalRouter` and by `PoolManager`:

- In `PoolManager`, as long as it is in a blockchain with `blockGasLimit >= 10` it won't cause issues. Which all the blockchain supported by Uniswap do. This is because it is only used in `ProtocolFees.sol` and with a default `bips == BLOCK_LIMIT_BPS == 100` and an `amount == block.gaslimit`.

- In `UniversalRouter`:
  - It is called in the `Payment.sol` library on `payPortion()`. Called when `Commands.PAY_PORTION` is specified in the dispatcher. See [here](https://github.com/Uniswap/universal-router/blob/a81e1ce93e5c87d106ccd936abc8517175f46572/contracts/modules/Payments.sol#L48).
  - And in the inherited `V4Router.sol` it is used like so: `_getFullCredit(currency).calculatePortion(bips)`. See [here](https://github.com/Uniswap/universal-router/blob/a81e1ce93e5c87d106ccd936abc8517175f46572/contracts/base/Dispatcher.sol#L143) and [here](https://github.com/Uniswap/v4-periphery/blob/151b28293733b9cd8a310babc15408fe1ba55c35/src/V4Router.sol#L80).

- Regarding `payPortion()` for small amounts users can't really transfer certain percentages trhough the router if desired as it will round down to 0. If the _`bips`_ are: _`bips < BPS_DENOMINATOR / amount`_. It rounds down to 0 and nothing is transferred. Bip calulations are not properly handled for dust amounts, and in case of ERC20s with 2 decimals, it will round down to 0 for significant amounts ([GUSD](https://etherscan.io/address/0x056fd409e1d7a124bd7017459dfea2f387b6d5cd#readContract) as example of token with 2 decimals). This is a problem a as a user can specify valid inputs and lets say if at the end of the interaction with `UniversalRouter` he wants to send back value to a recipient it will send just 0 and the value will remain in the `UniversalRouter` contract. It will remain there because the `payPortion()` uses as amount input the balance of `address(this) == UniversalRouter`.

- Regarding `_getFullCredit(currency).calculatePortion(bips)`, `_getFullCredit()` always returns a positive delta, which can be very small, or as said in case of 2 decimal tokens can be significant. If the desired `bips` have the same condition mentioned above, it will round down to 0 and the function will return 0. Then the following is called:

    `_take(currency, _mapRecipient(recipient), _getFullCredit(currency).calculatePortion(bips));`

    Making the user execute a take of 0 tokens. He loses nothing here, probably it will have an unexpeted revert due to delta != 0 from the `PoolManager` and that is all, yet it is incorrect router behaviour from valid specified inputs.

## Proof of Concept

You can copy paste this contract in `RemixIDE` and try the following input to see it rounds down:

- Amount: 20 -> Small dust amount or 20 cents of token from ERC20s with 2 decimals.
- Percentage: 499 -> 4.99%

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/// @title For calculating a percentage of an amount, using bips
contract BipsLibrary {
    uint256 internal constant BPS_DENOMINATOR = 10_000;

    /// @notice emitted when an invalid percentage is provided
    error InvalidBips();

    /// @param amount The total amount to calculate a percentage of
    /// @param bips The percentage to calculate, in bips
    function calculatePortion(uint256 amount, uint256 bips) public pure returns (uint256) {
        if (bips > BPS_DENOMINATOR) revert InvalidBips();
        return (amount * bips) / BPS_DENOMINATOR;
    }
}
```

## Recommendation

At `BipsLibrary.sol` make sure that `amount * percentage >= BPS_DENOMINATOR` to avoid rounding down to 0.

If so you might as well revert and for example if user wanted to execute 400 `bips` on an 20 `amount`, which result in `0.8` if calculated correctly, he will just have to input 10000 `bips` on 0.8 `amount`. So the operations are still posible and now the code will at least calculate things as expected and prevent the user from unexpected results.
