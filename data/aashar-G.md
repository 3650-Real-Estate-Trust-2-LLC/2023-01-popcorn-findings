1. Function state mutability of the below functions can be restricted to view
 a. _verifyCreatorOrOwner (L667) VaultController.sol
 b. _encodeAdapterData (L242) VaultController.sol
 c. _calcRewardsEnd (L351) MultiRewardStaking.sol

2.  Unused variable in Vault.sol, variable cached in function but not used - highWaterMark_ (L448) - Vault.sol

3. State variables that are constants may be marked as constant instead of immutable. VaultController.sol

4. State variables that are only set in the constructor should be declared immutable 
 a. L387, L717, L822, L535 - VaultController.sol
 b. L23, L24, L25 - DeploymentController.sol
