# LOW SEVERITY FINDINGS

## 1) MISCONFIGURED LOGIC/ REDUNDANT CODE  IN VAULTREGISTRY.SOL

The function getSubmitter() in interface IVaultRegistry.sol returns address 
 (https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVaultRegistry.sol#L27)

But in VaultRegistry.sol getSubmitter() returns the VaultMetadata
 (https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L75-L77)

And also getSubmitter() function is same as the getVault() function.
(https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L59-L61)

### RECOMMENDATION

Check the logic is correct in getSubmitter() function.
If it is correct  we don't require  getSubmitter() function as we can use getVault() function so that we can reduce redundant code.
If not  it will create issues in frontend.

