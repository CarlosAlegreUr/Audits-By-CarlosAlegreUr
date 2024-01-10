### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

### **`consolidatePendingStake()` should be public** 🔢

## **Summary 📌**

Currently, as `consolidatePendingStake()` is private, your stake is active only if someone does one of these actions: stakes, unstakes or liquidates. If these actions dont happen at least once every day the pending stakes won't be updated and users will have to execute one of this actions just to activate their old stakes.

These exposes users to an unnecessary risk of having their pending stake not activated and thus not earning rewards on it.

---

## **Vulnerability Details 🔍 && Impact 📈**

If these actions are happening every day there is no problem. But if not **users have to lose extra unnecessary money** activating their old stakes or they will be missing rewards.

The frecency is at least once every day because the `deadline` parameter is calculated like so:

```solidity
    function consolidatePendingStakes() private {
        uint256 deadline = block.timestamp - 1 days; // <---- 🟢 DEADLINE 
        for (int256 i = 0; uint256(i) < pendingStakes.length; i++) {
            PendingStake memory _stake = pendingStakes[uint256(i)];
            if (_stake.createdAt < deadline) { // <---- 🟢 Check for the deadline
               // Activate pending stake logic 
            }
        }
    }
```

---

## **Tools Used 🛠️**

- Manual audit.

---

## **Recommendations 🎯**

Make `consolidatePendingStakes()` public so anyone can call it and activate the stake without having to operate by increasing, decreasing stake or someone being liquidated on the system. This should not be a problem as the function does not change state in an uncontrolled way.

---