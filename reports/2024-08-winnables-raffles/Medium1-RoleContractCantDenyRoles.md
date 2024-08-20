### Summary

`Role` only allows granting roles, not revoking them. This is not intended behaviour as it is stated in the contest readme that `Role` can also deny roles.

### Root Cause

`status` parameter is ignored in `_setRole()` function. This makes it impossible to revoke a role.


### Internal pre-conditions

Does not apply.


### External pre-conditions

Does not apply.


### Attack Path

1. Admin grants a role to an address accidentally.
2. That role cant be revoked as expected.

### Impact

A core part of the protocol, its `Role` contract that manages roles, is incorrect. The consequences would be bad only if admin accidentally gives a role to a malicious actor as he would not be able to take that role away from him. 

It is true that for a great impact you need an admin error. Yet all stems and shows that an expected important functionality of the code is incomplete and incorrect thus I send it as `Medium`. Not even the admin can revoke role.

### PoC

As you can [see here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/Roles.sol#L31) the only bit value that can be set is 1.

`1 OR 1 = 1`
`1 OR 0 = 1`

Once a role is granted, it can't be denied. This is not expected as it can be read in the contest contract readme [here](https://github.com/sherlock-audit/2024-08-winnables-raffles?tab=readme-ov-file#q-for-permissioned-functions-please-list-all-checks-and-requirements-that-will-be-made-before-calling-the-function).

### Mitigation

Take into account the `status` parameter in `_setRole()`. Something like:

```diff
function _setRole(address user, uint8 role, bool status) internal virtual {
    uint256 roles = uint256(_addressRoles[user]);

-   _addressRoles[user] = bytes32(roles | (1 << role));
+    _addressRoles[user] = status 
+        ? bytes32(roles | (1 << role))  // Set the role bit to 1 (assign role)
+        : bytes32(roles & ~(1 << role)); // Clear the role bit (revoke role)

    emit RoleUpdated(user, role, status);
}
```