### [L-01] ```approve()``` and ```safeApprove()``` should be replaced with ```safeIncreaseAllowance()``` / ```safeDecreaseAllowance()```


#### Impact
```approve()``` & ```safeApprove()``` are deprecated and subject to a known front-running attack. Consider using  ```safeIncreaseAllowance()``` & ```safeDecreaseAllowance()``` instead.


#### Findings:
```
src/utils/MultiRewardStaking.sol:L271               rewardToken.safeApprove(address(escrow), type(uint256).max);

src/vault/Vault.sol:L80                 asset.approve(address(adapter_), type(uint256).max);

src/vault/Vault.sol:L604                 asset.approve(address(adapter), 0);

src/vault/Vault.sol:L610                 asset.approve(address(adapter), type(uint256).max);

src/vault/VaultController.sol:L171               asset.approve(address(target), initialDeposit);

src/vault/VaultController.sol:L456               IERC20(rewardsToken).approve(staking, type(uint256).max);

src/vault/adapter/yearn/YearnAdapter.sol:L54                 IERC20(_asset).approve(address(yVault), type(uint256).max);

src/vault/adapter/beefy/BeefyAdapter.sol:L80                 IERC20(asset()).approve(_beefyVault, type(uint256).max);

src/vault/adapter/beefy/BeefyAdapter.sol:L83                     IERC20(_beefyVault).approve(_beefyBooster, type(uint256).max);

```


### [N-01] Open TODOs


#### Impact
Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment.


#### Findings:
```
src/vault/adapter/abstracts/AdapterBase.sol:L516    // TODO use deterministic fee recipient proxy

```
