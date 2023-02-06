# QA report

## Summary
The contract code quality is average, and the documentation is incomplete and often confusing.

## List of non-critical findings:
1. `@notice` states that the function's "caller must be owner", but there is no `onlyOwner` modifier, and it must be open (check the docs). 
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/DeploymentController.sol#L59-L72
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/README.md?plain=1#L33
2. With `changeVaultAdapters()` and `changeVaultFees()` in the `VaultController` contract, it is possible to call `fallback` or selector functions `changeAdapter()` and `changeFees()` from any contract in the network without any checks on behalf of `AdminProxy`. It possibly can be a medium-severity issue.
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L331-L344
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L368-L381
3. Comment from another function.
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L369
4. Unused variable.
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L448
5. `@notice` states that the function's "caller must be owner", but there is `canCreate` modifier instead.
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L79
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L181
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L274
6. `@dev` states that there are 525600 minutes per year, but the function uses `SECONDS_PER_YEAR` constants, which is `365.25 days` or 525960 minutes.
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L422-L439
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L35