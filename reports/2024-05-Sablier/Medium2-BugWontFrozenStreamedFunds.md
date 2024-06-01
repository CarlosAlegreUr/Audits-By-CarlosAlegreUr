### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

## **Summary 📌**

The protocol has a bug protection mechanism which, in case of ever being exeuted, would allow stream `senders` to cancel streamed funds. Streamed funds are supposed to be streamed and not cancellable anymore.

---

## **Vulnerability Details 🔍**

In the case of a bug appearing, which I consider a valid scenario due to the very same codebae having this precautious measure for it, then streamed funds could be sent to the `sender` via the `cancel()` function.

This problem affects the `LockupLinear` contract and the `LockupDynamic` contract.

In both of these contracts there is a code similar to the following one:

```solidity
function _calculateStreamedAmount(uint256 streamId) internal view override returns (uint128) {

    // more code...

        // Although the streamed amount should never exceed the deposited amount, this condition is checked
        // without asserting to avoid locking funds in case of a bug. If this situation occurs, the withdrawn
        // amount is considered to be the streamed amount, and the stream is effectively frozen.
        if (streamedAmount.gt(depositedAmount)) {
            return _streams[streamId].amounts.withdrawn; // 🟢
        }
    // more code...
}
```

Then inside the `_cancel()`, `_calculateStreamedAmount()` is called and the funds sent to `sender` are computed like so: `senderAmount = amounts.deposited - streamedAmount;`. `streamedAmount` is the return value of `_calculateStreamedAmount()`.

But if this bug happens, the return value will be `withdrawn` amount, which does not necessarily correspond to the streamed amount. As the `recipient` could have withdrawn 0, but the funds are still streaming. Thus `senderAmount` would be `deposited - 0` and all deposited funds, which part of them are streamed and must not be cancellable, would be sent to the `sender`.

The reason why to return `withdrawn` amount is to protect the funds from being drained by a bug. This is because then the `withdrawableAmountOf()` [(see here)](https://github.com/Cyfrin/2024-05-Sablier/blob/main/v2-core/src/abstracts/SablierV2Lockup.sol#L543) would always return 0, effectively frozening them until `endTime` has passed. This proves that, yet frozen, the funds are still streaming. It is just that the `recipient` won't be able to claim them until `endTime` is reached in the case of `LockupLinear` for example.

---

## **Impact 📈**
        
In case of bug protection being activated, streamed funds can be stolen by the `sender`.

There are 3 parts on the codebase where this bug could be exploited:

- `LokcupLinear.sol` [here](https://github.com/Cyfrin/2024-05-Sablier/blob/main/v2-core/src/SablierV2LockupLinear.sol#L223).
- `LockupDynamic.sol` in 1 segment streams, [here](https://github.com/Cyfrin/2024-05-Sablier/blob/main/v2-core/src/SablierV2LockupDynamic.sol#L302).
- `LockupDynamic.sol` in >1 segment streams, [here](https://github.com/Cyfrin/2024-05-Sablier/blob/main/v2-core/src/SablierV2LockupDynamic.sol#L268). But notice, in this case the amount of streamed funds that can be stolen is limited to the amount of funds streamed on the segment affected by the bug. If time passes by and the next segment is activated and not affected by the `.gt()` condition, then the streamed funds of the affected segment won't be able to be cancelled anymore. This is due to this part of the code not always returning the `withdrawn` value.

---

## **Proof Of Concept (PoC)** 👨‍💻💻

The execution flow is pretty simple so I will link the key aspects of the execution:

1. A bug appears and then the `if (streamedAmount.gt(depositedAmount))` will enter returning `withdrawn` amount.

2. `sender` calls `cancel()` which calls `_cancel()`, which after [some checks](https://github.com/Cyfrin/2024-05-Sablier/blob/main/v2-core/src/abstracts/SablierV2Lockup.sol#L258) that do not check for frozen funds, calls `_calculateStreamedAmount()` [here](https://github.com/Cyfrin/2024-05-Sablier/blob/main/v2-core/src/abstracts/SablierV2Lockup.sol#L553).

3. You can see that until `senderAmount` is computed [here](https://github.com/Cyfrin/2024-05-Sablier/blob/main/v2-core/src/abstracts/SablierV2Lockup.sol#L571), there is only checks with 'if' statements thus the value returned by `_calculateStreamedAmount()` has not been altered. And, none of this checks will revert becase `streamedAmount` returned will be `withdrawn` amount. This is the only check concerning amounts [(this one)](https://github.com/Cyfrin/2024-05-Sablier/blob/main/v2-core/src/abstracts/SablierV2Lockup.sol#L559) and it will not revert because as said earlier, `recipient` can have withdrawn 0 so far.

4. And finally the `senderAmount` value is not altered nor checked against anything until it is eventually transfered [here](https://github.com/Cyfrin/2024-05-Sablier/blob/main/v2-core/src/abstracts/SablierV2Lockup.sol#L599).

Conclusion, in case of bug `sender` can steal streamed funds from `recipient`.

---

## **Tools Used 🛠️**

- Manual analysis.

---

## **Recommendations 🎯**

Frozen funds should not be cancellable. Add an extra return value to `_calculateStreamedAmount()`, a boolean flag indicating frozen funds. 

If entered in the if statement which means frozen, return `true` and inside the `_cancel()`, if returned `true` revert as streamed funds would be cancelled and that should not happen.

---
