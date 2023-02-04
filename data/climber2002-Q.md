## Certain events or errors are never used and can be deleted

1. The error [NotEndorsed(bytes32 templateKey)](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneFactory.sol#L32) defined in CloneFactory.sol is never used and can be deleted.

2. The event TemplateUpdated defined in [TemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L40) is never used

## Some functions can be restricted to view mutability

1. [StrategyBase.verifyAdapterSelectorCompatibility](https://github.com/climber2002/2023-01-popcorn/blob/1a6aea3e5e39de10befc61884d64a7dd8db4624e/src/vault/strategy/StrategyBase.sol#L12), from the method name it shouldn't change contract state

## fees_ param is not validated in Vault.initialize

The fees param is validated in [proposeFees](https://github.com/climber2002/2023-01-popcorn/blob/1a6aea3e5e39de10befc61884d64a7dd8db4624e/src/vault/Vault.sol#L527) but in [initialize](https://github.com/climber2002/2023-01-popcorn/blob/1a6aea3e5e39de10befc61884d64a7dd8db4624e/src/vault/Vault.sol#L88) the param is not validated.

We can extract a `_validateFees` internal function and call it in two places.

2. [VaultController. _encodeAdapterData](https://github.com/climber2002/2023-01-popcorn/blob/1a6aea3e5e39de10befc61884d64a7dd8db4624e/src/vault/VaultController.sol#L243)


