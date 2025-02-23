## Summary

The `AaveDIVAWrapper` assumes that WTokens are always redeemable, but this is not always true due to AAVE’s reserve states. AAVE reserves can become paused, fully utilized, or even dropped, affecting liquidity withdrawals.  

Users are currently not warned about these risks, leading to potential unexpected transaction failures and temporary loss of access to funds.

## Vulnerability Details

AAVE reserves can experience multiple states that block or delay withdrawals:

- Paused: No interactions (supply, borrow, repay, withdraw) can happen.
- High Utilization: If most of the liquidity is borrowed, withdrawals are delayed until liquidity is replenished.
- Dropped Asset: If AAVE removes an asset, all interactions with the reserve are permanently disabled.

Each of these states impacts WToken redemption, leading to failed transactions or forced delays.

### High Utilization Prevents WToken Unwrapping
- AAVE operates on a lending model where aTokens may not always be immediately withdrawable.
- If AAVE reaches high utilization, meaning most of the asset’s liquidity is borrowed, WTokens cannot be unwrapped, as unwrapping requires withdrawing the underlying asset from AAVE.
- The `_redeemWTokenPrivate()` function [here](https://github.com/Cyfrin/2025-01-diva/blob/main/contracts/src/AaveDIVAWrapperCore.sol#L470) fails if AAVE has no liquidity available.

Reference:  
[AAVE FAQ – How do I withdraw?](https://aave.com/faq)

### Paused Reserves Block All Liquidity Movement
- If a reserve is paused, withdrawals are completely blocked, meaning users cannot redeem WTokens.
- Additionally, no new liquidity can be supplied, meaning no new WTokens can be minted.
- This affects functions like `createContingentPool()` and `addLiquidity()`, which rely on `_handleTokenOperations()`, calling `supply()`.

Relevant AAVE code validation:  
- Supply validation logic (paused state blocks supply): [ValidationLogic.sol#L76](https://github.com/aave/aave-v3-origin/blob/main/src/core/contracts/protocol/libraries/logic/ValidationLogic.sol#L76)  
- Withdrawal validation logic (paused state blocks withdrawal): [ValidationLogic.sol#L105](https://github.com/aave/aave-v3-origin/blob/main/src/core/contracts/protocol/libraries/logic/ValidationLogic.sol#L105)

### Dropped Assets Permanently Break WToken Redemption
- If AAVE drops an asset (removes its reserve), all its state data is deleted (see [here](https://github.com/aave/aave-v3-origin/blob/main/src/core/contracts/protocol/libraries/logic/PoolLogic.sol#L154)), setting `isActive = false`.
- When this happens:
  - Supplies fail: See [ValidationLogic.sol#L75](https://github.com/aave/aave-v3-origin/blob/main/src/core/contracts/protocol/libraries/logic/ValidationLogic.sol#L75).
  - Withdrawals fail: See [ValidationLogic.sol#L105](https://github.com/aave/aave-v3-origin/blob/main/src/core/contracts/protocol/libraries/logic/ValidationLogic.sol#L105).

Users must be warned to withdraw their WTokens before an asset is dropped.

## Impact

- Users may be unable to redeem WTokens due to AAVE reserve states.
- Unexpected transaction failures can occur, blocking liquidity withdrawal.
- Integrators that depend on immediate WToken redemption can be disrupted, causing financial harm.
- If an asset is dropped, all WToken holders lose access to their underlying collateral permanently.

## Recommendations

Since these issues cannot be fixed as they depend on AAVE's code, they must be mitigated through explicit warnings.

> Note: I've grouped them all together because they all share the same nature (AAVE reserve state dependancies risks) and fix.
