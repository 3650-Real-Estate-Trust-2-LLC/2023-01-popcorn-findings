## [L-01] AVOID FLOATING PRAGMA
Files: All Files

see: https://swcregistry.io/docs/SWC-103

## [L-02] Use latest Solidity stable Pragma version

Files: All files 

#### Recomended Mitigation Steps
Consider using version 0.8.17 instead of the version 0.8.15.

## [NC-01] AVOID EMPTY BLOCKS
Avoid empty blocks or refactor the code so that empty blocks don't exist or emit an event.
Files: 

- [src/vault/Vault.sol#L477](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L477)
- [src/vault/VaultRegistry.sol#L21](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L21)
- [src/vault/adapter/abstracts/WithRewards.sol#L15](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/WithRewards.sol#L15)
- [src/vault/AdminProxy.sol#L16](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/AdminProxy.sol#L16)

## [NC-02] Some functions do not have NatSpec comments.
see: https://docs.soliditylang.org/en/v0.8.17/natspec-format.html

Files:

- [src/vault/Vault.sol#L716](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L716)
- [src/vault/Vault.sol#L664](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L664)

## [NC-03] Implement the TODO in the AdapterBase.sol contract.

The AdapterBase.sol contract has a TODO comment and TODO is an indication of unfinished work

#### Recommendation mitigation steps
Consider impelementing the TODO

## [NC-04] Use latest Openzeppelin library contract version.
File: package.json
The project uses openzeppelin library version 4.8.0 and the latest openzeppelin/contract version is 4.8.1 
It is always best practice to use the latest stable version of softwares as they will contain latest bugfixes and updates.



