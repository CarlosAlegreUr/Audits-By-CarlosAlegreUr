## Summary

If AAVE reaches its supply cap for a collateral asset, users will be unable to add liquidity to existing DIVA pools created through the `AaveDIVAWrapper`. This can disrupt ongoing campaigns, such as donation pools, by preventing additional contributions.

## Vulnerability Details

### Scenario:
1. A campaign is created using the `AaveDIVAWrapper`, which supplies collateral to AAVE and mints WTokens.
2. Over time, as the campaign proves successful, additional liquidity is expected to be added.
3. Before more liquidity is supplied, AAVE reaches its **supply cap** for that collateral asset.
4. Since AAVE no longer accepts new deposits, `aave.supply()` will revert, preventing WToken minting.
5. Without WTokens, adding liquidity to the existing DIVA pool becomes impossible, creating a DOS for liquidity provition to the pool.

### Realistic Use Case:
- A donation campaign starts with initial funding.
- As trust builds, more liquidity is added to the pool through `addLiquidity()`.
- If AAVE’s supply cap is hit in the meantime, additional liquidity can no longer be added, blocking further donations.
- The only alternative would be to create a **new DIVA pool**, requiring previous liquidity to be removed and re-supplied, which incurs additional protocol fees. Also this should be made interacting directly with DIVA protocol, which is costly and time consuming developer-wise.

## Impact

- Users will be unable to add liquidity to DIVA pools when AAVE reaches its supply cap of the collateral asset for aToken.  
- Campaigns relying on incremental liquidity will be **disrupted** without warning.  
- Users will be forced to remove and recreate pools, leading to unnecessary and unfair **extra fees** in the DIVA Protocol.  
- Since WTokens can only be minted via the wrapper's `_handleTokenOperations()` (see [here](https://github.com/Cyfrin/2025-01-diva/blob/main/contracts/src/AaveDIVAWrapperCore.sol#L440)), which is the function calling `supply()` (reverts if max supply reached, see [here](https://github.com/aave/aave-v3-origin/blob/main/src/core/contracts/protocol/libraries/logic/ValidationLogic.sol#L86)). Directly adding liquidity through DIVA contracts is not an option. As the collateral in the exisitng DIVA pool is the WToken and no-one can get that token anymore.  

## Recommendations

A fallback mechanism should be implemented in the wrapper to handle cases where AAVE reaches its supply cap:

1. **Detect when AAVE’s supply cap is reached** using existing AAVE contract functions.  
2. **Redirect collateral to an escrow contract** instead of attempting to supply it to AAVE.  
3. **Allow WTokens to be minted regardless**, using the escrowed collateral instead of aTokens.  
4. Once AAVE allows new deposits again, a function should allow the escrowed collateral to be supplied to AAVE.  

This ensures that donation campaigns and other use cases remain uninterrupted, even when AAVE reaches temporary supply limits.
