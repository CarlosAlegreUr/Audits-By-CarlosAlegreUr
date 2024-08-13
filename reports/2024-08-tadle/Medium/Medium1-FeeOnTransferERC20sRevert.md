## **Vulnerability Details 🔍 && Impact 📈**

Protocol claims to support any ERC20 token yet some ERC20 have the fee on token transfer functioanlity which leads to a DOS on critical functions. 

Fee on token transfer means that if `from` sends `amount` to `to`, `to` will receive `amount - fee`.

Thus if using this tokens, in `_transfer()` on `TokenManager.sol` the following checks will revert:

```solidity
if (fromBalanceAft != fromBalanceBef - _amount) {
           revert TransferFailed();
       }

if (toBalanceAft != toBalanceBef + _amount) {
    revert TransferFailed();
}
```

See them [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/TokenManager.sol#L255), and [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/TokenManager.sol#L259).

This causes a DOS of `_transfer()` when used with fee on transfer tokens. The impact is high as `_transfer()` is a function that gets called in critical parts of the protool like in the `withdraw()` or `tillIn()`, functions. Functions used for moving value around in and out of the system.

Also, some tokens might have the ability of turning on and off the fees, if they turn on the fees after people have made maker-taker operations in Tadle, the collateral will be stuck as it won't be able to be withdrawn.

---

## **Recommendations 🎯**

As collateral tokens have to be whitelisted by the team ([see here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/TokenManager.sol#L30)), add a special flag that indicates if some whitelisted collateral includes fee on transfer. In those cases you can use a special check that allows for a little bit of difference in the prev and post balance checks.

---