## Function state mutability can be restricted to view
VaultController.sol - [242](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L242), [667](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L667)

## Variable `highWaterMark_` is initialised but never read from, consider using it instead of storage variable `highWaterMark`
[Vault.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L448-L459)