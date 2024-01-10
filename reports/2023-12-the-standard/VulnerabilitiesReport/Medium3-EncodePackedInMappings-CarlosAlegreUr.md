### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

### **`abi.encodePacked()` should not be used for mappings' IDs** 🔢

## **Summary 📌**

> ℹ️ **Note** 📘: Aderyn caught the use of `abi.encodePacked()` in some files of the system that are out of socope. But there are files in-scope that suffer from this problem and in a different way. In the Aderyn cases the use of `abi.encodePacked()` is highligthed when generatin NFT string type metadata in `NFTMetadataGenerator` and the other contracts assosiated with it (all out of scope).

In the `LiquidtaionPool` contract `abi.encodePaked()` is used as unique keys for the `rewards` mapping. As `abi.encodePaked()` can have the same output for differnet inputs, this ID can someday clash.

---

## **Vulnerability Details 🔍 && Impact 📈**

The clash of IDs would overwrite the `rewards` mapping for a user and thus it would overwrite and potentially lose any rewards he might have accumulated from a different asset.

---

## **Tools Used 🛠️**

- Manual audit.

---

## **Recommendations 🎯**

Instead of using as ID key `abi.encodePacked()` use the `keccak256()` hash function:

```diff
- rewards[abi.encodePacked(userAddress, asset.token.symbol)]
+ rewards[keccack256(userAddress, asset.token.symbol)]
```

> 🚧 **Note** ⚠️: On top of all this the current values that make up the ID could potentially not
> be unique enough. Tokens can have the same symbol and this limits the amount of assets your protocol
> could use in the future without and ID clash. As current assets do not have shared symbols this is not a problem for now but consider adding something extra for the ID calculation like the asset's contract address.

---