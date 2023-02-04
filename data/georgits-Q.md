## Use latest Solidity version with a stable pragma statement
All solidity files

## Missing a check if the clone has already been registered
CloneRegistry.sol - [41](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L41)

## NatSpec must be updated
DeploymentController.sol - [60](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L60)

## Remove unused imports
YearnAdapter.sol - [6](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L6)(IStrategy, IAdapter)

BeefyAdapter.sol - [6](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L6)(IStrategy)

## Use of ecrecover is susceptible to signature malleability, consider using [OpenZeppelin's ECDSA library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol)
MultiRewardStaking.sol - [459](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L459)

Vault.sol - [678](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L678)

AdapterBase.sol - [646](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L646)