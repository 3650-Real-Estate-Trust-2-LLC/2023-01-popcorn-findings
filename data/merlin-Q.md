# Non-critical
## Useless error and event

- TemplateRegistry smart contract has unused event:
[TemplateRegistry.sol#L40](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L40)
- CloneFactory smart contract has unused error:
[CloneFactory.sol#L32](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneFactory.sol#L32)

# Low
## Shadowing Local Variables

Possible incorrect use of variables are at stake which may have bad side effects to the contract if implemented incorrectly.
According to the Slither-analyzer documentation (https://github.com/crytic/slither/wiki/Detector-Documentation#local-variable-shadowing), shadowing local variables is naming conventions found in two or more variables that are similar. Although they do not pose any immediate risk to the contract, incorrect usage of the variables is possible and can cause serious issues if the developer does not pay close attention.

Consider to change all variables with names `owner` that match with `OwnedUpgradeable.owner `.
It is recommended that the naming of the following variables should be changed slightly to avoid any confusion:
- [Vault smart contract](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L214)
- [MultiRewardStaking smart contract](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L124)
- etc

