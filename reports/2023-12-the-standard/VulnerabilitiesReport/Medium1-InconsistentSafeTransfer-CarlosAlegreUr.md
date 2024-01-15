### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

### **Inconsistent usage of `safeTransfer()` can make user miss rewards** 🔢

## **Summary 📌**

Inconsistent usage of `transfer()` and `safeTransfer()` trhoughout the codebase can lead to user missing rewards.

---

## **Vulnerability Details 🔍 && Impact 📈**

When dealing with the whitelisted assets (those the team has deemed as valid collateral) they sometimes use `safeTransfer()` and other just `transfer()`. For example when dealing with collateral removal opearations they use the safe version but when users claim rewards they use the traditional transfer.

Transfer is used in another function but the main problem resides here: `claimRewards()` in `LiquidationPool`. The way this function is programmed makes it so if the tx sending the rewards fails silently then the user wont be able to claim them again as the state that tracks them would be deleted.

Follow the numbers 1️⃣ in the code for a clearer explanation.

```solidity
    function claimRewards() external {
        // code...
            uint256 _rewardAmount = rewards[abi.encodePacked(msg.sender, _token.symbol)];
            //🔴 3️⃣ if deleted on failed tx then _rewardAmount == 0 so transfer wont be executed 
            // until next liquidation. Even so as explained in 2️⃣, the amount in the failed tx liquidation 
            // will be lost.
            if (_rewardAmount > 0) {
                // 🔴2️⃣ deleting rewards. Even if rewards is incremented again in another 
                // liquidation, the value in the silent fail will alredy be deleted and never claimed.
                delete rewards[abi.encodePacked(msg.sender, _token.symbol)]; 
                if (_token.addr == address(0)) {
                    // code...
                } else {
                    // 🔴1️⃣ this tx could fail sliently. 
                    // For example with return false on fail-transfer tokens
                    IERC20(_token.addr).transfer(msg.sender, _rewardAmount); 
                }
            }
        }
    }
```

>  ℹ️ **Note** 📘: `forwardRemainingRewards()` in `LiquidationPoolManager` also uses `transfer()`. But here I didn't see any problem some tx failing momentarily should not be a problem as rewards can be forwarded next time a liquidation occures.

---

## **Tools Used 🛠️**

- Manual review.

---

## **Recommendations 🎯**

Use `safeTransfer()` for at least the `claimRewards()` function in the `LiquidationPool` contract.

---