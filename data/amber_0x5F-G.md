## immutable variable would save gas
In DeploymentController.sol , there are 3 variables :  cloneFactory, cloneRegistry, templateRegistry will not change again after contract deployed.
At line 23-25 : https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L23

## constant variable would save more gas than immutable one.
At line 36-40 :  https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L36