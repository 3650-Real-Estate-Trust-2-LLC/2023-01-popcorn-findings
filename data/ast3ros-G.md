# [GAS-01] Add checking assets == 0 and return assets to avoid complex calculation

Change to below to avoid `mulDiv` and `totalAssets` calculation when assets = 0
        return
            (supply == 0 || assets == 0)
                ? assets
                : assets.mulDiv(supply, totalAssets(), Math.Rounding.Down);

Instance:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L297-L300

# [GAS-02] Use memory variable instead of state variable to save gas

highWaterMark_ variable is unused.
Use local variable highWaterMark_ instead of state variable highWaterMark to calculate accrued fee.

Instance:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L448

# [GAS-03] Verify input before initiate variable to save gas

Reading state variable `deploymentController` should move below `_verifyToken` and `_verifyAdapterConfiguration`. 
If the verifications fails, we can avoid reading the state variable:
    IDeploymentController _deploymentController = deploymentController;

Instance:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L98

# [GAS-04] State variables should be cached in stack variables rather than re-reading them from storage

Instance:
deploymentController: 
- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L837
- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L840

# [GAS-05] Using functions in ERC4626Upgradeable instead of override to save deployment gas

Function `deposit` and `mint` could be inherited from ERC4626Upgradeable instead of override again in AdapterBase contract to save gas when deploying
https://github.com/code-423n4/2023-01-popcorn/blob/dcdd3ceda3d5bd87105e691ebc054fb8b04ae583/src/vault/adapter/abstracts/AdapterBase.sol#L110-L122
https://github.com/code-423n4/2023-01-popcorn/blob/dcdd3ceda3d5bd87105e691ebc054fb8b04ae583/src/vault/adapter/abstracts/AdapterBase.sol#L129-L141