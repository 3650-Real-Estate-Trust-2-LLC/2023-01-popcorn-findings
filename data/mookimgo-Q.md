# 1. VaultRegistry's getSubmitter function has wrong return value

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IVaultRegistry.sol#L27

The interface IVaultRegistry has defined `getSubmitter` to return an address.

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L75-L77

But this code is returning `VaultMetadata`, which is wrong. In this implementation, this function is working exactly same as getVault, which should not be expected.
