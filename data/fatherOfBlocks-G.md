**src/vault/adapter/yearn/YearnAdapter.sol**
- L151 - A subtraction is made that in line 150 the underflow is checked, therefore you can avoid performing the check and use unchecked.


**src/vault/adapter/beefy/BeefyAdapter.sol**
- L154/155/175/176 - A variable is created in memory but it is only used once, therefore the variable in memory could be eliminated and used directly where it is used.


**src/utils/MultiRewardStaking.sol**
- L144/145/198/200/305/311/308/314/336/339/424/427/431 - A variable is created in memory but it is only used once, therefore the variable could be deleted in memory and use it directly where it is used.


**src/vault/Vault.sol**
- L483/490 - A variable is created in memory but it only ends up being used once, so you could delete the variable in memory and use it directly where it is used.


**src/vault/VaultController.sol**
- L524/527 - A variable is created in memory but it only ends up being used once, so you could delete the variable in memory and use it directly where it is used.

- L526/527/634/636/647/649/682/683/684/706/707/708 - A parameter in storage is called twice, therefore gas could be saved if a variable is created in memory with this value.


