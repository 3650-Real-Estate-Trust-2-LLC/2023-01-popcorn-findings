# 1. VaultRegistry's allVaults can be marked as private to save gas

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L34

As function `getRegisteredAddresses` has returned `allVaults` ( all items returned by a single call ), there's no need to mark `allVaults` as public.

There's no usage of `allVaults(uint256)` in other contract, so marking as private has no impact.
Off-chain consumers can always fetch the array items by eth_getStorage.

# 2. CloneRegistry's clones and allClones can be marked as private to save gas

public clones's behavior is same as   `function getClonesByCategoryAndId(bytes32 templateCategory, bytes32 templateId)`

public allClones is not needed as there's getAllClones