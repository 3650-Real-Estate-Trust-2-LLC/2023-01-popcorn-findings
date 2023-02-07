## [GAS-01] x = x + y cost less gas than x +=y for state variables. Same for x = x -y.

Files

- [src/vault/adapter/abstracts/AdapterBase.sol#L158](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L158)

## [GAS-02] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for-loop and while-loops.

Files:

- [src/vault/VaultController.sol#L319](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L319)
- [src/vault/VaultController.sol#L357](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L357)
- [src/vault/VaultController.sol#L374](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L374)


