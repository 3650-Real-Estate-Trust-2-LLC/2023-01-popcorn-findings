### Unspecific Compiler Version Pragma

#### Impact
Issue Information: 
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

Example
🤦 Bad:

pragma solidity ^0.8.0;
🚀 Good:

pragma solidity 0.8.4;

#### Findings:
```
2023-01-popcorn/src/interfaces/IEIP165.sol::2 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/IMultiRewardEscrow.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/IMultiRewardStaking.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/IOwned.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/IPausable.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/IPermit.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/external/IWETH.sol::4 => pragma solidity ^0.8.0;
2023-01-popcorn/src/interfaces/external/uni/IUniswapRouterV2.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/vault/IAdapter.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/vault/IAdminProxy.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/vault/ICloneFactory.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/vault/ICloneRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/vault/IDeploymentController.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/vault/IERC4626.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/vault/IPermissionRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/vault/IStrategy.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/vault/ITemplateRegistry.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/vault/IVault.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/vault/IVaultController.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/vault/IVaultRegistry.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn/src/interfaces/vault/IWithRewards.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/utils/EIP165.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/utils/MultiRewardEscrow.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/utils/MultiRewardStaking.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/utils/Owned.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/utils/OwnedUpgradeable.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/vault/AdminProxy.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/vault/CloneFactory.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/vault/CloneRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/vault/DeploymentController.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/vault/PermissionRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/vault/TemplateRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/vault/Vault.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/vault/VaultController.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn/src/vault/VaultRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/vault/adapter/abstracts/OnlyStrategy.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/vault/adapter/abstracts/WithRewards.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/vault/adapter/beefy/BeefyAdapter.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/vault/adapter/beefy/IBeefy.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/vault/adapter/yearn/IYearn.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn/src/vault/adapter/yearn/YearnAdapter.sol::4 => pragma solidity ^0.8.15;
```
#### Tools used
Manual VS code review