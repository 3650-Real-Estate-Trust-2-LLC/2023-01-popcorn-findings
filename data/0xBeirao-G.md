### Function visibility should be just enough 

These fonction must be set **external** to save gas on call : 
- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L124
- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L200

### Cached **highWaterMark_** not used in **accruedPerformanceFee()**

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L447-L460

Use the cached **highWaterMark_** instead of the stored one in the return statement.


### Increments/decrements can be unchecked in for-loops

Consider wrapping with an unchecked block here (around 25 gas saved per instance):

``` js 
- for (uint256 i; i < numIterations; i++) {
+ for (uint256 i; i < numIterations;) {
 // ...  
+   unchecked { ++i; }
}  
```
The risk of overflow is non-existent for uint256 here.

-> This gas optimisation is present in all **for loops** of the code base.

### Use the correct decoration 

These variables are not initialized is the constructor. Use **constant** instead of **immutable** to save gas.

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L36-L39

### _registerVault parameters optimization 

**address vault** is not used

from :
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L390

to : 
```solidity
function _registerVault(VaultMetadata calldata metadata) internal {...}
```

### Put **indexed** parameters in all event from **_Registry** type of contracts

The emitted events are not indexed, making off-chain scripts such as front-ends of dApps difficult to filter the events efficiently.

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L38-L40
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L28
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L36
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneRegistry.sol#L33

**Addresses** and **bytes32** should be **indexed** at these lines.
