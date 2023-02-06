**src/vault/Vault.sol**
- It is somewhat confusing that the directory is vault/vault, therefore the name of the folder should be something else that encompasses all the contracts that are inside.

- L448/453/455 - The variable in memory highWaterMark_ is created, but it is never used. Inside the return the parameter highWaterMark is used twice, but highWaterMark_ should be used.

- L470 - The InsufficientWithdrawalAmount error is created, but it is never used, therefore it should be eliminated.


**src/vault/VaultController.sol**
- L11 - Template is imported, but it is never used, therefore it should be removed.


**src/vault/adapter/abstracts/AdapterBase.sol**
- L40/103 - StrategySetupFailed and ZeroAmount errors are created, but never used, therefore they should be removed.


**src/vault/adapter/yearn/IYearn.sol**
- L13/15/19/21 - Within the interface, VaultAPI, multiple functions are created that are never used, therefore they should be removed. These include pricePerShare, totalAssets, depositLimit, and token.


**src/interfaces/vault/IERC4626.sol**
- L44 - Within the interface, IERC4626, a function is created, mint(), which is never used, therefore it should be removed.


**src/vault/adapter/beefy/IBeefy.sol**
- L13/22/24/30/38/40/46 - Within the IBeefy contract, multiple interfaces are created that have functions that are never used, among them: withdrawAll(), earn(), getPricePerFullShare(), earned(address) ), periodFinish(), rewardPerToken(), and exit(). Therefore, since they are not used, they should be removed.


**src/vault/adapter/yearn/YearnAdapter.sol**
- L6 - ERC4626, IStrategy and IAdapter are imported, but they are never used, so they should be removed.


**src/utils/MultiRewardEscrow.sol**
- L200 - The NoFee error is created, but it is never used, therefore it should be removed.


**src/vault/adapter/beefy/BeefyAdapter.sol**
- L6 - IStrategy is imported, but it is never used, so it should be removed.


**src/utils/MultiRewardStaking.sol**
- L226 - The NotSubmitter error is created, but it is never used, so it should be removed.

- L359 - A division is performed by an input, therefore it could be a zero and that should be checked with a validation, because if the function is reversed the user should know why.


**src/vault/adapter/abstracts/WithRewards.sol**
- L12/13/15 - The contract has several functions that are not implemented, therefore instead of a simple contract, it should be an abstract.


**src/vault/CloneFactory.sol**
- L32 - The NotEndorsed error is created, but it is never used, therefore it should be removed.
