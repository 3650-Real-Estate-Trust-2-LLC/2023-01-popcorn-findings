1. Remove adminProxy and maybe use an Upgradeable Proxy instead if you want to make it easy to replace the VaultController. There'll be no need to change owner of all the contracts that are directly owned by VaultController when using an upgradeable proxy for VaultController. Removing adminProxy makes it easier to read and understand the code for VaultController.

2. Change these immutable variables to constants: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L36-L40

3. Since CloneRegistry and CloneFactory are small, they can be combined into DeploymentController. Apart from the deploy function, DeploymentController's functions are just TemplateRegistry functions and ownership-related functions. Once combined, there'll be significantly less code that needs to be read less contract interactions to keep track of to understand the system.

4. Add support for setQuitPeriod and setFeeRecipent functions to VaultController. Those Vault functions can't be called if VaultController owns the Vault:
	1. https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L629-L636
	2. https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L553-L559
5. Maintain same interface of functions and the same behavior where possible for all ERC-4626 contracts. For example, Vault has deposit(to) but Adapter doesn't. AdapterBase uses underlyingBalance state variable but Vault doesn't.
6. Use safeTransfer here https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L182
7. IDeploymentController needs to be updated since the interface doesn't match the actual functions in DeploymentController. For example, addClone, PermissionRegistry and getTemplate are not functions in DeploymentController - https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IDeploymentController.sol#L38-L40 
8. DeploymentController.nominateNewDependencyOwner can't be called by VaultController. Need to first change owner of adminProxy to an EOA so that function can be called and DeploymentController can be replaced.
9. VaultController can't nominateNewOner for all the contracts that AdminProxy owns. Need to first change owner of adminProxy to an EOA so that nominateNewOwner can be called for all those contracts.
10. Fees in Vault are much smaller and Vault creators will get a much lower fee than expected. Setting deposit fees to 99% only gets 5000 from 50000e18 deposit. Fees computation should be updated so that 10% deposit fee really gets 10% of the deposit. Once that change is applied, limit the fees to a max of probably 10% or less - https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L521-L537