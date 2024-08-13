## Vulnerability Details 🔍 && Impact 📈

The `approve()` of `CapitalPool.sol` expects the token address (see [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/CapitalPool.sol#L28)) to later call approve in that address. Yet `TokenManager.sol` in its `_transfer()` calls it like so: `ICapitalPool(_capitalPoolAddr).approve(address(this));`. See [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/TokenManager.sol#L247).

Clearly `address(this)` is not a token address and trying to call `approve()` on it will fail as. This might have been a small oversight as exactly in the `if` statement on top of the approve call they are using the actual token address, see [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/TokenManager.sol#L245).

Yet small oversight imapct is high and will happen in every call to `_transfer()` than involves `from==CapitalPool`, a.k.a. -> withdrawing funds out of the system. The current state of the system withdrawals is unusable as no matter what collateral you use the contracts will revert when trying to transfer from `CapitalPool` as the approval will always revert. Thus the funds inserted to the pool are stuck there.

---

## Recommendations 🎯

The approve should be called passing the `_token` function parameter from `_transfer()`.

Like so:

```diff solidity
        if (
            _from == _capitalPoolAddr &&
            IERC20(_token).allowance(_from, address(this)) == 0x0
        ) {
-           ICapitalPool(_capitalPoolAddr).approve(address(this));
+           ICapitalPool(_capitalPoolAddr).approve(_token);
        }
```

---