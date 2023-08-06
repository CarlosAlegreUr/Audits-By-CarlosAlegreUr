## 📌 **Summary**

In file `DSCEngine.sol`, a hardcoded number in the code can be converted into an existing named constant for better readability.

---

## 🔍 **Vulnerability Details**

In the line 311 of `DSCEngine.sol`, contrary to the rest of the code, a number is being used directly without a named constant.

---

## 📈 **Impact**

Slightly worse readability.

> **Notice** ℹ️: I ran `forge snapshot` in the original code and the one with this "renamed" number. As expected, there is neither a gas benefit nor a drawback in its addition. The tests consumed an equivalent amount of gas.

---

## 🛠️ **Tools Used**

- Manual audit.
- Forge snapshot.
- Custom bash scripts to analyze snapshots' results.

---

## 🎯 **Recommendations**

Given that a named constant with the identical value already exists and its name is contextually relevant, it's recommended to replace the direct number with this constant:

```
(line 331)  
// From  
return (collateralAdjustedForThreshold * 1e18) / totalDscMinted;  

// To  
return (collateralAdjustedForThreshold * PRECISION) / totalDscMinted;
```
