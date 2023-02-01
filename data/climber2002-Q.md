## Summary
Certain events or errors are never used and can be deleted

## Details
1. The error [NotEndorsed(bytes32 templateKey)](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneFactory.sol#L32) defined in CloneFactory.sol is never used and can be deleted.

2. The event TemplateUpdated defined in [TemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L40) is never used