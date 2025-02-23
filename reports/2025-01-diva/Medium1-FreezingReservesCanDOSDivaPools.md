## Summary

AAVE reserves can enter a `frozen` state, preventing new deposits and borrowing while still allowing withdrawals and repayments. This can impact protocols like DIVA that rely on the ability to supply liquidity, making certain functions unusable.

## Vulnerability Details

AAVE reserves can transition to a frozen state, where only withdrawals and repayments are allowed, but new liquidity cannot be supplied. 

This directly affects the `AaveDIVAWrapper`, as it relies on `aave.supply()` not reverting to mint WTokens. If the underlying AAVE reserve is frozen, `supply()` will revert, preventing:
- Creating new pools using `createContingentPool()`.
- Adding liquidity to existing pools via `addLiquidity()`.

Both functions call `_handleTokenOperations()`, which in turn calls `supply()`.

See AAVE's supply validation logic, where freezing prevents deposits:  
[ValidationLogic.sol#L77](https://github.com/aave/aave-v3-origin/blob/main/src/core/contracts/protocol/libraries/logic/ValidationLogic.sol#L77)

## Impact

If AAVE freezes a reserve used by DIVA pools, new liquidity can no longer be supplied, making affected pools unusable. 

For example, in a donation campaign scenario, an initial pool may be created with limited funds, with plans to add more liquidity over time as the campaing gains trust. If AAVE freezes the reserve in the meantime, additional liquidity cannot be supplied, potentially halting the campaign indefinitely, even beyond the `expiryTime`.

Even interacting directly with DIVA contracts would not resolve the issue, as WTokens must be minted through the wrapper, which relies on `supply()` succesfully executing, see that the only mint executes right after, [here](https://github.com/Cyfrin/2025-01-diva/blob/main/contracts/src/AaveDIVAWrapperCore.sol#L440). This means that as long as AAVE’s reserve remains frozen, additional liquidity cannot be provided because no-one can get new WTokens, the ones used in the existing DIVA pools as collateral.

## Recommendations

A fallback mechanism should be implemented to allow liquidity to be supplied even when AAVE freezes a reserve, as the lack of it will DOS valid use-cases:

- If the system detects that AAVE has frozen a reserve during `addLiqudity()`, collateral should be redirected to an escrow contract. This prevents the valid use-case of a pool being created while the reserve was not freezed yet needing to add liquidity later at a time where reserve might be frozen.
- A function should also be created to deposit the escrowed funds into AAVE once the reserve is unfrozen.

This ensures that no donation campaing gets unnecessarily DOS by AAVE freezing reserves.
