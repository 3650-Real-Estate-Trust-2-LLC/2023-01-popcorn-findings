## Use of named returns for local variables saves gas
You can have further advantages in term of gas cost by simply using named return values as temporary local variable.

For instance, the code block below may be refactored as follows:

[File: WithRewards.sol#L21-L23](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/WithRewards.sol#L21-L23)

```diff
-  function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {
+  function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool status_) {
-    return interfaceId == type(IWithRewards).interfaceId || interfaceId == type(IAdapter).interfaceId;
+    status_ = interfaceId == type(IWithRewards).interfaceId || interfaceId == type(IAdapter).interfaceId;
  }
```
## Unused custom error
Consider removing the custom error below that has been declared but never used in the contract to save gas on contract deployment:

[File: CloneFactory.sol#L32](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneFactory.sol#L32)

```diff
-  error NotEndorsed(bytes32 templateKey);
```
## `||` costs less gas than its equivalent `&&`
Rule of thumb: `(x && y)` is `(!(!x || !y))`

Even with the 10k Optimizer enabled: `||`, OR conditions cost less than their equivalent `&&`, AND conditions.

As an example, the instance below may be refactored as follows:

[File: PermissionRegistry.sol#L43](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L43)

```diff
-      if (newPermissions[i].endorsed && newPermissions[i].rejected) revert Mismatch();
+      if (!(!newPermissions[i].endorsed || !newPermissions[i].rejected)) revert Mismatch();
```
## Unchecked SafeMath saves gas
"Checked" math, which is default in ^0.8.0 is not free. The compiler will add some overflow checks, somehow similar to those implemented by `SafeMath`. While it is reasonable to expect these checks to be less expensive than the current `SafeMath`, one should keep in mind that these checks will increase the cost of "basic math operation" that were not previously covered. This particularly concerns variable increments in for loops. When no arithmetic overflow/underflow is going to happen, `unchecked { ++i ;}` to use the previous wrapping behavior further saves gas just as in the for loop below as an example:

[File: PermissionRegistry.sol#L42-L48](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L42-L48)

```diff
-        for (uint256 i = 0; i < len; i++) {
+        for (uint256 i = 0; i < len;) {
              if (newPermissions[i].endorsed && newPermissions[i].rejected) revert Mismatch();

              [ ... ]

+            unchecked {
+                i++;
+            }
          }
```