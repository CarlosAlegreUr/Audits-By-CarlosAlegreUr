## Summary

DIVA's rounding mechanics can lead to a situation where **position tokens are burned without returning collateral**. Users and developers interacting with `AaveDIVAWrapper` may be unaware of this issue and must be warned on the wrapper's docs. As the wrapper is a system on itself to simplify DIVA interactions with AAVE features, users must be able to work safely only relying on the wrapper's code.

## Vulnerability Details

### How Rounding Can Cause Collateral Loss:
- DIVA's core documentation states that **for small `_amount` values, position tokens may be burned, but no collateral is returned** when redeeming them.
- This is due to mathematical rounding in the redemption process, which can effectively result in **a user losing their tokens without receiving any compensation**.

**Reference:**  
DIVA Protocol’s official documentation highlights this issue:  
> *"For small values of `_amount`, the position token may get burnt, but no collateral returned due to rounding. It is recommended to handle such cases on the frontend side accordingly."*  
([DIVA Protocol Docs](https://github.com/divaprotocol/diva-protocol-v1/blob/main/DOCUMENTATION.md#redeempositiontoken))

### AAVE’s Zero Withdrawal Issue:
- If a user calls `AaveDIVAWrapper::redeemPositionToken()` and the rounding results in **0 collateral being withdrawn**, AAVE's `withdraw()`reverts because the transaction is passed a 0 amount value. The 0 value comes from this substraction [here](https://github.com/Cyfrin/2025-01-diva/blob/main/contracts/src/AaveDIVAWrapperCore.sol#L295). And the withdraw call is the one on the `_withdrawWTokenPrivate()` [here](https://github.com/Cyfrin/2025-01-diva/blob/main/contracts/src/AaveDIVAWrapperCore.sol#L470). 
- The relevant not 0 check can be seen in [AAVE’s smart contract code](https://github.com/aave/aave-v3-origin/blob/main/src/core/contracts/protocol/libraries/logic/ValidationLogic.sol#L101C5-L101C49).

## Impact

- Users may **lose tokens without receiving collateral** due to rounding.  
- Calls to `redeemPositionToken()` can **unexpectedly revert**, breaking certain integrations.  
- Developers using `AaveDIVAWrapper` can be unaware of this behavior since they should not need to read the core DIVA documentation to use the wrapper.  

## Recommendations

1. **Explicit User Warnings**  
   - The `AaveDIVAWrapper` should provide clear documentation about DIVA's rounding mechanics.  
   - Developers should be informed that **small token amounts may be burned without collateral return**.  

2. **Frontend Handling**  
   - Since DIVA recommends handling rounding at the frontend, **UI/UX designers should implement safeguards** to prevent users from redeeming small token amounts.  
  
By making these risks explicit and providing proper frontend handling, users can avoid unintended token loss and failed transactions. Translating DIVA's core security to the wrapper.
