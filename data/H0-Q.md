## [N-01] Use latest Solidity Version

Context:
All contracts

Description:
For security, it is best practice to use the latest Solidity version.

For the security fix list in the versions;

https://github.com/ethereum/solidity/blob/develop/Changelog.md

Recommendation:
Old version of Solidity is used , newer version can be used (0.8.17)

## [N-02] Unused local variable, apparently distinctive from other use cases in the code

[Vault.sol L449](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L448)