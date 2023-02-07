getSubmitter function implementation has different return type to it's definiation in interface.
 
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L75

```
function getSubmitter(address vault) external view returns (VaultMetadata memory) {
```

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVaultRegistry.sol#L27

```
function getSubmitter(address vault) external view returns (address);
```