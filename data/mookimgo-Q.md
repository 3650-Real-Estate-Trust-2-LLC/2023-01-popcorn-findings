# 1. VaultRegistry's getSubmitter function has wrong return value

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IVaultRegistry.sol#L27

The interface IVaultRegistry has defined `getSubmitter` to return an address.

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L75-L77

But this code is returning `VaultMetadata`, which is wrong. In this implementation, this function is working exactly same as getVault, which should not be expected.

# 2. AdapterBases FEE_RECIPIENT is a invalid constant

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L563

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L517

adapter's performance fee is sent to this invalid address `0x4444`, this should be changed to a meaningful address.

No change can be made for this FEE_RECIPIENT, consider adding a function to change it, or mark it as constant or immutable (setting it in the constructor)