

### [L01] Missing checks for `address(0x0)` when assigning values to `address` state variables


#### Findings:
```
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::31 => feeRecipient = _feeRecipient;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::53 => escrow = _escrow;
2023-01-popcorn-main/src/vault/DeploymentController.sol::41 => cloneFactory = _cloneFactory;
2023-01-popcorn-main/src/vault/DeploymentController.sol::42 => cloneRegistry = _cloneRegistry;
2023-01-popcorn-main/src/vault/DeploymentController.sol::43 => templateRegistry = _templateRegistry;
2023-01-popcorn-main/src/vault/Vault.sol::558 => feeRecipient = _feeRecipient;
2023-01-popcorn-main/src/vault/Vault.sol::633 => quitPeriod = _quitPeriod;
2023-01-popcorn-main/src/vault/VaultController.sol::61 => adminProxy = _adminProxy;
2023-01-popcorn-main/src/vault/VaultController.sol::62 => vaultRegistry = _vaultRegistry;
2023-01-popcorn-main/src/vault/VaultController.sol::63 => permissionRegistry = _permissionRegistry;
2023-01-popcorn-main/src/vault/VaultController.sol::64 => escrow = _escrow;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::80 => strategyConfig = _strategyConfig;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::81 => harvestCooldown = _harvestCooldown;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::585 => underlyingBalance = _underlyingBalance();
2023-01-popcorn-main/test/utils/mocks/ClonableWithInitData.sol::9 => val = _val;
2023-01-popcorn-main/test/utils/mocks/MockERC4626.sol::45 => asset = _asset;
```

### [L02] `safeApprove()` is deprecated

#### Impact
Deprecated in favor of safeIncreaseAllowance() and safeDecreaseAllowance()
#### Findings:
```
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::271 => rewardToken.safeApprove(address(escrow), type(uint256).max);
2023-01-popcorn-main/test/utils/Faucet.sol::99 => IERC20(dai).safeApprove(address(cDai), type(uint256).max);
2023-01-popcorn-main/test/utils/Faucet.sol::100 => cDai.safeApprove(address(crvCompPool), type(uint256).max);
2023-01-popcorn-main/test/utils/Faucet.sol::101 => IERC20(crv).safeApprove(address(crvCvxCrvPool), type(uint256).max);
2023-01-popcorn-main/test/utils/Faucet.sol::102 => IERC20(dai).safeApprove(address(crvAavePool), type(uint256).max);
2023-01-popcorn-main/test/utils/Faucet.sol::103 => IERC20(dai).safeApprove(address(triPool), type(uint256).max);
2023-01-popcorn-main/test/utils/Faucet.sol::104 => IERC20(usdt).safeApprove(address(crv3CryptoPool), type(uint256).max);
2023-01-popcorn-main/test/utils/Faucet.sol::105 => IERC20(wbtc).safeApprove(address(crvSBtcPool), type(uint256).max);
2023-01-popcorn-main/test/utils/Faucet.sol::106 => IERC20(crvSBtcLP).safeApprove(address(crvIbBtcPool), type(uint256).max);
```




### [L03] approve should be replaced with safeApprove or safeIncreaseAllowance() / safeDecreaseAllowance()

#### Impact
approve is subject to a known front-running attack. Consider using safeApprove instead:
#### Findings:
```
2023-01-popcorn-main/src/vault/Vault.sol::80 => asset.approve(address(adapter_), type(uint256).max);
2023-01-popcorn-main/src/vault/Vault.sol::604 => asset.approve(address(adapter), 0);
2023-01-popcorn-main/src/vault/Vault.sol::610 => asset.approve(address(adapter), type(uint256).max);
2023-01-popcorn-main/src/vault/VaultController.sol::171 => asset.approve(address(target), initialDeposit);
2023-01-popcorn-main/src/vault/VaultController.sol::456 => IERC20(rewardsToken).approve(staking, type(uint256).max);
2023-01-popcorn-main/src/vault/adapter/beefy/BeefyAdapter.sol::80 => IERC20(asset()).approve(_beefyVault, type(uint256).max);
2023-01-popcorn-main/src/vault/adapter/beefy/BeefyAdapter.sol::83 => IERC20(_beefyVault).approve(_beefyBooster, type(uint256).max);
2023-01-popcorn-main/src/vault/adapter/yearn/YearnAdapter.sol::54 => IERC20(_asset).approve(address(yVault), type(uint256).max);
2023-01-popcorn-main/test/MultiRewardEscrow.t.sol::50 => token1.approve(address(escrow), 10 ether);
2023-01-popcorn-main/test/MultiRewardEscrow.t.sol::51 => token2.approve(address(escrow), 10 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::90 => stakingToken.approve(address(staking), aliceUnderlyingAmount);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::132 => stakingToken.approve(address(staking), 0.5e18);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::140 => stakingToken.approve(address(staking), 0.5e18);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::163 => stakingToken.approve(address(staking), aliceShareAmount);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::205 => stakingToken.approve(address(staking), 0.5e18);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::250 => stakingToken.approve(address(staking), 5 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::312 => stakingToken.approve(address(staking), 5 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::385 => stakingToken.approve(address(staking), 5 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::412 => stakingToken.approve(address(staking), 1 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::413 => staking.approve(bob, 1 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::418 => stakingToken.approve(address(staking), 1 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::419 => staking.approve(alice, 1 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::496 => stakingToken.approve(address(staking), 2 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::535 => stakingToken.approve(address(staking), 5 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::594 => rewardsToken.approve(address(staking), 10 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::605 => rewardsToken.approve(address(staking), 10 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::613 => rewardToken1.approve(address(staking), 10 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::673 => rewardToken1.approve(address(staking), 10 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::693 => rewardToken1.approve(address(staking), 20 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::708 => rewardToken1.approve(address(staking), 1 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::724 => stakingToken.approve(address(staking), 1 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::726 => stakingToken.approve(address(staking), 1 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::794 => rewardToken1.approve(address(staking), 10 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::816 => rewardToken1.approve(address(staking), 10 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::838 => rewardToken1.approve(address(staking), 10 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::879 => stakingToken.approve(address(staking), 5 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::901 => rewardToken1.approve(address(staking), 5 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::905 => stakingToken.approve(address(staking), 1 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::931 => stakingToken.approve(address(staking), 5 ether);
2023-01-popcorn-main/test/vault/Vault.t.sol::191 => asset.approve(address(vault), aliceassetAmount);
2023-01-popcorn-main/test/vault/Vault.t.sol::236 => asset.approve(address(vault), 0.5e18);
2023-01-popcorn-main/test/vault/Vault.t.sol::244 => asset.approve(address(vault), 0.5e18);
2023-01-popcorn-main/test/vault/Vault.t.sol::267 => asset.approve(address(vault), aliceShareAmount);
2023-01-popcorn-main/test/vault/Vault.t.sol::312 => asset.approve(address(vault), 0.5e18);
2023-01-popcorn-main/test/vault/Vault.t.sol::320 => asset.approve(address(vault), 0.5e18);
2023-01-popcorn-main/test/vault/Vault.t.sol::391 => asset.approve(address(vault), 4000);
2023-01-popcorn-main/test/vault/Vault.t.sol::398 => asset.approve(address(vault), 7001);
2023-01-popcorn-main/test/vault/Vault.t.sol::552 => asset.approve(address(vault), 1e18);
2023-01-popcorn-main/test/vault/Vault.t.sol::555 => asset.approve(address(vault), 1e18);
2023-01-popcorn-main/test/vault/Vault.t.sol::601 => asset.approve(address(vault), amount);
2023-01-popcorn-main/test/vault/Vault.t.sol::620 => asset.approve(address(vault), amount);
2023-01-popcorn-main/test/vault/Vault.t.sol::625 => asset.approve(address(vault), amount);
2023-01-popcorn-main/test/vault/Vault.t.sol::655 => asset.approve(address(vault), depositAmount);
2023-01-popcorn-main/test/vault/Vault.t.sol::684 => asset.approve(address(vault), depositAmount);
2023-01-popcorn-main/test/vault/Vault.t.sol::833 => asset.approve(address(vault), depositAmount);
2023-01-popcorn-main/test/vault/Vault.t.sol::915 => asset.approve(address(vault), depositAmount * 3);
2023-01-popcorn-main/test/vault/Vault.t.sol::953 => asset.approve(address(vault), depositAmount * 2);
2023-01-popcorn-main/test/vault/VaultController.t.sol::285 => rewardToken.approve(address(controller), 10 ether);
2023-01-popcorn-main/test/vault/VaultController.t.sol::376 => rewardToken.approve(address(controller), 10 ether);
2023-01-popcorn-main/test/vault/VaultController.t.sol::524 => rewardToken.approve(address(controller), 10 ether);
2023-01-popcorn-main/test/vault/VaultController.t.sol::578 => rewardToken.approve(address(controller), 10 ether);
2023-01-popcorn-main/test/vault/VaultController.t.sol::682 => rewardToken.approve(address(controller), 10 ether);
2023-01-popcorn-main/test/vault/VaultController.t.sol::748 => rewardToken.approve(address(controller), 10 ether);
2023-01-popcorn-main/test/vault/VaultController.t.sol::866 => rewardToken.approve(address(controller), 10 ether);
2023-01-popcorn-main/test/vault/VaultController.t.sol::869 => asset.approve(address(controller), 1 ether);
2023-01-popcorn-main/test/vault/VaultController.t.sol::985 => asset.approve(address(controller), 1 ether);
2023-01-popcorn-main/test/vault/VaultController.t.sol::1370 => rewardToken2.approve(address(controller), 10 ether);
2023-01-popcorn-main/test/vault/VaultController.t.sol::1656 => rewardToken.approve(address(controller), 10 ether);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::109 => asset.approve(address(adapter), amount);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::189 => asset.approve(address(adapter), defaultAmount);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::201 => asset.approve(address(adapter), defaultAmount);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::224 => asset.approve(address(adapter), maxAssets);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::234 => asset.approve(address(adapter), maxAssets);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::316 => adapter.approve(alice, type(uint256).max);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::340 => adapter.approve(alice, type(uint256).max);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::122 => asset.approve(address(vault), amount);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::131 => asset.approve(address(vault), reqAssets);
2023-01-popcorn-main/test/vault/integration/beefy/BeefyAdapter.t.sol::79 => asset.approve(address(adapter), defaultAmount);
2023-01-popcorn-main/test/vault/integration/yearn/YearnAdapter.t.sol::71 => asset.approve(address(adapter), defaultAmount);
```



### [L04] Return values of `transfer()`/`transferFrom()` not checked

#### Impact
Not all IERC20 implementations revert() when there’s a failure in transfer()/transferFrom(). The function signature has a boolean
 return value and they indicate errors that way instead. By not checking
 the return value, operations that should have been marked as failed may 
potentially go through without actually making a payment
#### Findings:
```
2023-01-popcorn-main/src/vault/VaultController.sol::457 => IERC20(rewardsToken).transferFrom(msg.sender, address(adminProxy), amount);
2023-01-popcorn-main/test/utils/mocks/MockAdapter.sol::61 => IERC20(asset()).transfer(address(mockYieldFarm), assets);
```




### [L05] Unspecific Compiler Version Pragma

#### Impact
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.
#### Findings:
```
2023-01-popcorn-main/src/interfaces/IEIP165.sol::2 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/IMultiRewardEscrow.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/IMultiRewardStaking.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IAdapter.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IAdminProxy.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/ICloneFactory.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/ICloneRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IDeploymentController.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IERC4626.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IPermissionRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IStrategy.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/ITemplateRegistry.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IVault.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IVaultController.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IVaultRegistry.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IWithRewards.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/utils/EIP165.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/AdminProxy.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/CloneFactory.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/CloneRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/DeploymentController.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/PermissionRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/TemplateRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/Vault.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/VaultController.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/VaultRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/adapter/abstracts/OnlyStrategy.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/adapter/abstracts/WithRewards.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/adapter/beefy/BeefyAdapter.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/adapter/beefy/IBeefy.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/adapter/yearn/IYearn.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/adapter/yearn/YearnAdapter.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/MultiRewardEscrow.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/utils/EnhancedTest.sol::2 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/utils/Faucet.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/utils/mocks/ClonableWithInitData.sol::1 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/utils/mocks/ClonableWithoutInitData.sol::1 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/utils/mocks/MockAdapter.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/utils/mocks/MockERC20.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/utils/mocks/MockERC4626.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/utils/mocks/MockStrategy.sol::1 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/CloneFactory.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/CloneRegistry.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/DeploymentController.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/PermissionRegistry.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/TemplateRegistry.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/Vault.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/VaultController.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/VaultRegistry.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/abstract/ITestConfigStorage.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/abstract/PropertyTest.prop.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/beefy/BeefyAdapter.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/beefy/BeefyTestConfigStorage.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/beefy/BeefyVault.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/yearn/YearnAdapter.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/yearn/YearnTestConfigStorage.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/yearn/YearnVault.t.sol::4 => pragma solidity ^0.8.15;
```


### [L06] SOLMATE’S SAFETRANSFERLIB DOESN’T CHECK WHETHER THE ERC20 CONTRACT EXISTS


#### Impact
Solmate’s SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn’t exist (yet).
#### Findings:
```
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::100 => token.safeTransferFrom(msg.sender, address(this), amount);
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::111 => token.safeTransfer(feeRecipient, fee);
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::165 => escrow.token.safeTransfer(escrow.account, claimable);

2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::200 => rewardToken.safeTransfer(user, payout);
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::259 => rewardToken.safeTransferFrom(msg.sender, address(this), amount);
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::271 => rewardToken.safeApprove(address(escrow), type(uint256).max);
```

Recommended Mitigation Steps
Add a contract exist control in functions;

```
pragma solidity ^0.8.15;
function isContract(address _addr) private returns (bool isContract) {
    isContract = _addr.code.length > 0;
}
```



### [L07] REMOVE UNUSED CODE

#### Impact
This code is not used in the project, remove it or add event-emit;


#### Findings:
```
2023-01-popcorn-main/src/utils/EIP165.sol::7 => function supportsInterface(bytes4 interfaceId) public view virtual returns (bool) {}
2023-01-popcorn-main/src/vault/AdminProxy.sol::16 => constructor(address _owner) Owned(_owner) {}
2023-01-popcorn-main/src/vault/CloneFactory.sol::23 => constructor(address _owner) Owned(_owner) {}
2023-01-popcorn-main/src/vault/CloneRegistry.sol::22 => constructor(address _owner) Owned(_owner) {}
2023-01-popcorn-main/src/vault/PermissionRegistry.sol::20 => constructor(address _owner) Owned(_owner) {}
2023-01-popcorn-main/src/vault/TemplateRegistry.sol::24 => constructor(address _owner) Owned(_owner) {}
2023-01-popcorn-main/src/vault/Vault.sol::477 => {}
2023-01-popcorn-main/src/vault/VaultRegistry.sol::21 => constructor(address _owner) Owned(_owner) {}
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::258 => function _totalAssets() internal view virtual returns (uint256) {}
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::265 => function _underlyingBalance() internal view virtual returns (uint256) {}
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::276 => {}
2023-01-popcorn-main/src/vault/adapter/abstracts/WithRewards.sol::13 => function rewardTokens() external view virtual returns (address[] memory) {}
2023-01-popcorn-main/src/vault/adapter/abstracts/WithRewards.sol::15 => function claim() public virtual onlyStrategy {}
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::88 => function createAdapter() public virtual {}
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::92 => function increasePricePerShare(uint256 amount) public virtual {}
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::101 => function verify_totalAssets() public virtual {}
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::103 => function verify_adapterInit() public virtual {}
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::155 => function test__rewardsTokens() public virtual {}
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::549 => function testClaim() public virtual {}
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::93 => function increasePricePerShare(uint256 amount) public virtual {}
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::102 => function createAdapter() public virtual {}
```



### Non-Critical Issues

### [N01] Adding a return statement when the function defines a named return variable, is redundant



#### Findings:
```
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::56 => return selectedEscrows;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::60 => return _name;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::64 => return _symbol;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::68 => return _decimals;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::99 => return assets;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::103 => return shares;
2023-01-popcorn-main/src/vault/CloneRegistry.sol::66 => return allClones;
2023-01-popcorn-main/src/vault/TemplateRegistry.sol::116 => return templateCategories;
2023-01-popcorn-main/src/vault/Vault.sol::101 => return _decimals;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::95 => return _decimals;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::121 => return shares;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::140 => return assets;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::184 => return shares;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::203 => return assets;
2023-01-popcorn-main/src/vault/adapter/beefy/BeefyAdapter.sol::92 => return _name;
2023-01-popcorn-main/src/vault/adapter/beefy/BeefyAdapter.sol::101 => return _symbol;
2023-01-popcorn-main/src/vault/adapter/beefy/BeefyAdapter.sol::140 => return _rewardTokens;
2023-01-popcorn-main/src/vault/adapter/beefy/BeefyAdapter.sol::184 => return assets;
2023-01-popcorn-main/src/vault/adapter/yearn/YearnAdapter.sol::63 => return _name;
2023-01-popcorn-main/src/vault/adapter/yearn/YearnAdapter.sol::72 => return _symbol;
```



### [N02] constants should be defined rather than using magic numbers



#### Findings:
```
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::108 => uint256 fee = Math.mulDiv(amount, feePerc, 1e18);
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::197 => uint256 escrowed = rewardAmount.mulDiv(uint256(escrowInfo.escrowPercentage), 1e18, Math.Rounding.Down);
2023-01-popcorn-main/src/vault/adapter/yearn/YearnAdapter.sol::25 => uint256 constant DEGRADATION_COEFFICIENT = 10**18;
2023-01-popcorn-main/test/MultiRewardEscrow.t.sol::38 => token1 = new MockERC20("Mock Token1", "TKN1", 18);
2023-01-popcorn-main/test/MultiRewardEscrow.t.sol::39 => token2 = new MockERC20("Mock Token2", "TKN2", 18);
2023-01-popcorn-main/test/MultiRewardEscrow.t.sol::260 => uint256 expectedFee1 = Math.mulDiv(10 ether, 1e14, 1e18);
2023-01-popcorn-main/test/MultiRewardEscrow.t.sol::261 => uint256 expectedFee2 = Math.mulDiv(10 ether, 1e16, 1e18);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::38 => stakingToken = new MockERC20("Staking Token", "STKN", 18);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::40 => rewardToken1 = new MockERC20("RewardsToken1", "RTKN1", 18);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::41 => rewardToken2 = new MockERC20("RewardsToken2", "RTKN2", 18);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::127 => staking.deposit(1e18, address(this));
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::131 => stakingToken.mint(address(this), 0.5e18);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::132 => stakingToken.approve(address(staking), 0.5e18);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::133 => assertEq(stakingToken.allowance(address(this), address(staking)), 0.5e18);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::135 => staking.deposit(1e18, address(this));
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::139 => stakingToken.mint(address(this), 0.5e18);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::140 => stakingToken.approve(address(staking), 0.5e18);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::142 => staking.deposit(0.5e18, address(this));
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::144 => staking.withdraw(1e18, address(this), address(this));
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::148 => staking.withdraw(1e18, address(this), address(this));
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::200 => staking.mint(1e18, address(this));
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::204 => stakingToken.mint(address(this), 0.5e18);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::205 => stakingToken.approve(address(staking), 0.5e18);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::207 => staking.deposit(0.5e18, address(this));
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::209 => staking.redeem(1e18, address(this), address(this));
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::213 => staking.redeem(1e18, address(this), address(this));
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::230 => keccak256(abi.encode(PERMIT_TYPEHASH, owner, address(0xCAFE), 1e18, 0, block.timestamp))
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::235 => staking.permit(owner, address(0xCAFE), 1e18, block.timestamp, v, r, s);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::237 => assertEq(staking.allowance(owner, address(0xCAFE)), 1e18);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::619 => staking.addRewardToken(iRewardToken1, 0.1 ether, 10 ether, true, 10000000, 100, 20);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::628 => assertEq(uint256(escrowPercentage), 10000000);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::654 => staking.addRewardToken(iRewardToken1, 0, 0, true, 10000000, 100, 20);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::679 => staking.addRewardToken(iRewardToken1, 0.1 ether, 10 ether, true, 10000000, 100, 0);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::695 => staking.addRewardToken(iRewardToken1, 0.1 ether, 10 ether, true, 10000000, 100, 0);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::698 => staking.addRewardToken(iRewardToken1, 0.1 ether, 10 ether, true, 10000000, 100, 0);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::702 => staking.addRewardToken(IERC20(address(stakingToken)), 0.1 ether, 10 ether, true, 10000000, 100, 0);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::710 => staking.addRewardToken(iRewardToken1, 0, 1 ether, true, 10000000, 100, 20);
2023-01-popcorn-main/test/utils/Faucet.sol::80 => CrvCompPool public crvCompPool = CrvCompPool(0xA2B47E3D5c44877cca798226B7B8118F9BFb7A56);
2023-01-popcorn-main/test/vault/CloneFactory.t.sol::45 => emit Deployment(address(0x4f81992FCe2E1846dD528eC0102e6eE1f61ed3e2));
2023-01-popcorn-main/test/vault/CloneFactory.t.sol::77 => emit Deployment(address(0x4f81992FCe2E1846dD528eC0102e6eE1f61ed3e2));
2023-01-popcorn-main/test/vault/Vault.t.sol::24 => uint256 ONE = 1e18;
2023-01-popcorn-main/test/vault/Vault.t.sol::44 => asset = new MockERC20("Mock Token", "TKN", 18);
2023-01-popcorn-main/test/vault/Vault.t.sol::101 => assertEq(newVault.decimals(), 18);
2023-01-popcorn-main/test/vault/Vault.t.sol::135 => MockERC20 newAsset = new MockERC20("New Mock Token", "NTKN", 18);
2023-01-popcorn-main/test/vault/Vault.t.sol::151 => MockERC20 newAsset = new MockERC20("New Mock Token", "NTKN", 18);
2023-01-popcorn-main/test/vault/Vault.t.sol::236 => asset.approve(address(vault), 0.5e18);
2023-01-popcorn-main/test/vault/Vault.t.sol::237 => assertEq(asset.allowance(address(this), address(vault)), 0.5e18);
2023-01-popcorn-main/test/vault/Vault.t.sol::243 => asset.mint(address(this), 0.5e18);
2023-01-popcorn-main/test/vault/Vault.t.sol::244 => asset.approve(address(vault), 0.5e18);
2023-01-popcorn-main/test/vault/Vault.t.sol::246 => vault.deposit(0.5e18, address(this));
2023-01-popcorn-main/test/vault/Vault.t.sol::248 => vault.withdraw(1e18, address(this), address(this));
2023-01-popcorn-main/test/vault/Vault.t.sol::252 => vault.withdraw(1e18, address(this), address(this));
2023-01-popcorn-main/test/vault/Vault.t.sol::311 => asset.mint(address(this), 1e18);
2023-01-popcorn-main/test/vault/Vault.t.sol::312 => asset.approve(address(vault), 0.5e18);
2023-01-popcorn-main/test/vault/Vault.t.sol::313 => assertEq(asset.allowance(address(this), address(vault)), 0.5e18);
2023-01-popcorn-main/test/vault/Vault.t.sol::315 => vault.mint(1e18, address(this));
2023-01-popcorn-main/test/vault/Vault.t.sol::319 => asset.mint(address(this), 0.5e18);
2023-01-popcorn-main/test/vault/Vault.t.sol::320 => asset.approve(address(vault), 0.5e18);
2023-01-popcorn-main/test/vault/Vault.t.sol::322 => vault.deposit(0.5e18, address(this));
2023-01-popcorn-main/test/vault/Vault.t.sol::324 => vault.redeem(1e18, address(this), address(this));
2023-01-popcorn-main/test/vault/Vault.t.sol::328 => vault.redeem(1e18, address(this), address(this));
2023-01-popcorn-main/test/vault/Vault.t.sol::455 => assertEq(vault.convertToAssets(vault.balanceOf(alice)), aliceassetAmount + (mutationassetAmount / 3) * 1);
2023-01-popcorn-main/test/vault/Vault.t.sol::457 => assertEq(vault.convertToAssets(vault.balanceOf(bob)), bobassetAmount + (mutationassetAmount / 3) * 2);
2023-01-popcorn-main/test/vault/Vault.t.sol::631 => uint256 withdrawAmount = (amount / 10) * 9;
2023-01-popcorn-main/test/vault/Vault.t.sol::818 => MockERC20 newAsset = new MockERC20("New Mock Token", "NTKN", 18);
2023-01-popcorn-main/test/vault/Vault.t.sol::856 => assertEq(asset.balanceOf(address(newAdapter)), depositAmount * 2);
2023-01-popcorn-main/test/vault/Vault.t.sol::857 => assertEq(newAdapter.balanceOf(address(vault)), depositAmount * 2);
2023-01-popcorn-main/test/vault/Vault.t.sol::913 => asset.mint(alice, depositAmount * 3);
2023-01-popcorn-main/test/vault/Vault.t.sol::915 => asset.approve(address(vault), depositAmount * 3);
2023-01-popcorn-main/test/vault/Vault.t.sol::916 => vault.deposit(depositAmount * 2, alice);
2023-01-popcorn-main/test/vault/Vault.t.sol::951 => asset.mint(alice, depositAmount * 2);
2023-01-popcorn-main/test/vault/Vault.t.sol::953 => asset.approve(address(vault), depositAmount * 2);
2023-01-popcorn-main/test/vault/VaultController.t.sol::163 => asset = new MockERC20("Test Token", "TKN", 18);
2023-01-popcorn-main/test/vault/VaultController.t.sol::166 => rewardToken = new MockERC20("RewardToken", "RTKN", 18);
2023-01-popcorn-main/test/vault/VaultController.t.sol::1367 => MockERC20 rewardToken2 = new MockERC20("Reward Token2", "RTKN2", 18);
2023-01-popcorn-main/test/vault/VaultRegistry.t.sol::14 => MockERC20 asset = new MockERC20("ERC20", "TEST", 18);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::63 => defaultAmount = 10**IERC20Metadata(address(asset_)).decimals();
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::65 => raise = defaultAmount * 100_000;
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::66 => maxAssets = defaultAmount * 1000;
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::242 => uint256 reqAssets = (adapter.previewMint(adapter.previewWithdraw(amount)) * 10) / 9;
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::253 => uint256 reqAssets = (adapter.previewMint(amount) * 10) / 9;
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::303 => uint256 reqAssets = (adapter.previewMint(adapter.previewWithdraw(amount)) * 10) / 8;
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::327 => uint256 reqAssets = (adapter.previewMint(amount) * 10) / 9;
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::434 => _mintFor(defaultAmount * 3, bob);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::483 => increasePricePerShare(raise * 100);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::485 => uint256 gain = ((adapter.convertToAssets(1e18) - adapter.highWaterMark()) * adapter.totalSupply()) / 1e18;
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::565 => keccak256(abi.encode(PERMIT_TYPEHASH, owner, address(0xCAFE), 1e18, 0, block.timestamp))
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::572 => assertEq(adapter.allowance(owner, address(0xCAFE)), 1e18, "allowance");
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::38 => uint256 constant DEPOSIT_FEE = 50 * 1e14;
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::39 => uint256 constant WITHDRAWAL_FEE = 50 * 1e14;
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::40 => uint256 constant MANAGEMENT_FEE = 200 * 1e14;
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::41 => uint256 constant PERFORMANCE_FEE = 2000 * 1e14;
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::63 => defaultAmount = 10**IERC20Metadata(address(asset_)).decimals();
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::64 => maxDeposit = defaultAmount * 10_000;
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::186 => increasePricePerShare(amount * 1000);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::241 => increasePricePerShare(amount * 1000);
2023-01-popcorn-main/test/vault/integration/beefy/BeefyAdapter.t.sol::139 => setPermission(address(0x3af3563Ba5C68EB7DCbAdd2dF0FcE4CC9818e75c), true, false);
2023-01-popcorn-main/test/vault/integration/beefy/BeefyAdapter.t.sol::145 => address(0x3af3563Ba5C68EB7DCbAdd2dF0FcE4CC9818e75c)
2023-01-popcorn-main/test/vault/integration/beefy/BeefyAdapter.t.sol::151 => abi.encode(address(0x3af3563Ba5C68EB7DCbAdd2dF0FcE4CC9818e75c), _beefyBooster)
2023-01-popcorn-main/test/vault/integration/beefy/BeefyTestConfigStorage.sol::19 => BeefyTestConfig(0xF79BF908d0e6d8E7054375CD80dD33424B1980bf, 0x69C28193185CFcd42D62690Db3767915872bC5EA)
2023-01-popcorn-main/test/vault/integration/yearn/YearnAdapter.t.sol::85 => 10**IERC20Metadata(address(adapter)).decimals(),
2023-01-popcorn-main/test/vault/integration/yearn/YearnTestConfigStorage.sol::17 => testConfigs.push(YearnTestConfig(0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48));
```


### [N03] Variable names that consist of all capital letters should be reserved for `const`/`immutable `variables

#### Impact
If the variable needs to be different based on which class it comes from, a view/pure function should be used instead (e.g. like [this](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/76eee35971c2541585e05cbf258510dda7b2fbc6/contracts/token/ERC20/extensions/draft-IERC20Permit.sol#L59)).
#### Findings:
```
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::71 => uint256 internal nonce;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::438 => uint256 internal INITIAL_CHAIN_ID;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::439 => bytes32 internal INITIAL_DOMAIN_SEPARATOR;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::491 => function computeDomainSeparator() internal view virtual returns (bytes32) {
2023-01-popcorn-main/src/vault/Vault.sol::38 => uint8 internal _decimals;
2023-01-popcorn-main/src/vault/Vault.sol::657 => uint256 internal INITIAL_CHAIN_ID;
2023-01-popcorn-main/src/vault/Vault.sol::658 => bytes32 internal INITIAL_DOMAIN_SEPARATOR;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::241 => uint256 internal underlyingBalance;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::258 => function _totalAssets() internal view virtual returns (uint256) {}
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::625 => uint256 internal INITIAL_CHAIN_ID;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::626 => bytes32 internal INITIAL_DOMAIN_SEPARATOR;
2023-01-popcorn-main/src/vault/adapter/beefy/BeefyAdapter.sol::24 => string internal _name;
2023-01-popcorn-main/src/vault/adapter/beefy/BeefyAdapter.sol::25 => string internal _symbol;
2023-01-popcorn-main/src/vault/adapter/yearn/YearnAdapter.sol::21 => string internal _name;
2023-01-popcorn-main/src/vault/adapter/yearn/YearnAdapter.sol::22 => string internal _symbol;
2023-01-popcorn-main/test/utils/mocks/MockAdapter.sol::24 => MockYieldFarm internal mockYieldFarm;
2023-01-popcorn-main/test/utils/mocks/MockERC20.sol::8 => uint8 internal _decimals;
2023-01-popcorn-main/test/utils/mocks/MockERC4626.sol::18 => uint8 internal _decimals;
2023-01-popcorn-main/test/vault/integration/abstract/PropertyTest.prop.sol::10 => uint256 internal _delta_;
2023-01-popcorn-main/test/vault/integration/abstract/PropertyTest.prop.sol::12 => address internal _asset_;
2023-01-popcorn-main/test/vault/integration/abstract/PropertyTest.prop.sol::13 => address internal _vault_;
2023-01-popcorn-main/test/vault/integration/beefy/BeefyTestConfigStorage.sol::14 => BeefyTestConfig[] internal testConfigs;
2023-01-popcorn-main/test/vault/integration/yearn/YearnTestConfigStorage.sol::13 => YearnTestConfig[] internal testConfigs;
```



### [N04] Open TODOs

#### Impact
Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment
#### Findings:
```
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::516 => // TODO use deterministic fee recipient proxy
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::544 => // Accrual doesnt start until someone deposits -- TODO does this change some of the rewardsEnd and rewardsSpeed assumptions?
```



### [N05] Numeric values having to do with time should use time units for readability

#### Impact
There are units for seconds, minutes, hours, days, and weeks

#### Findings:
```
2023-01-popcorn-main/src/vault/Vault.sol::35 => uint256 constant SECONDS_PER_YEAR = 365.25 days;
2023-01-popcorn-main/src/vault/Vault.sol::619 => uint256 public quitPeriod = 3 days;
2023-01-popcorn-main/src/vault/Vault.sol::630 => if (_quitPeriod < 1 days || _quitPeriod > 7 days)
2023-01-popcorn-main/src/vault/VaultController.sol::793 => if (newCooldown > 1 days) revert InvalidHarvestCooldown(newCooldown);
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::502 => if (newCooldown >= 1 days) revert InvalidHarvestCooldown(newCooldown);
2023-01-popcorn-main/test/vault/Vault.t.sol::79 => vm.warp(block.timestamp + 3 days);
2023-01-popcorn-main/test/vault/Vault.t.sol::116 => assertEq(newVault.quitPeriod(), 3 days);
2023-01-popcorn-main/test/vault/Vault.t.sol::743 => vm.warp(block.timestamp + 3 days);
2023-01-popcorn-main/test/vault/Vault.t.sol::766 => // Didnt respect 3 days before propsal and change
2023-01-popcorn-main/test/vault/Vault.t.sol::845 => vm.warp(block.timestamp + 3 days);
2023-01-popcorn-main/test/vault/Vault.t.sol::873 => // Didnt respect 3 days before propsal and change
2023-01-popcorn-main/test/vault/Vault.t.sol::882 => uint256 newQuitPeriod = 1 days;
2023-01-popcorn-main/test/vault/Vault.t.sol::893 => vault.setQuitPeriod(1 days);
2023-01-popcorn-main/test/vault/Vault.t.sol::897 => vault.setQuitPeriod(23 hours);
2023-01-popcorn-main/test/vault/Vault.t.sol::901 => vault.setQuitPeriod(8 days);
2023-01-popcorn-main/test/vault/VaultController.t.sol::313 => 2 days,
2023-01-popcorn-main/test/vault/VaultController.t.sol::314 => 1 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::374 => controller.setHarvestCooldown(1 days);
2023-01-popcorn-main/test/vault/VaultController.t.sol::406 => 2 days,
2023-01-popcorn-main/test/vault/VaultController.t.sol::407 => 1 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::447 => assertEq(IAdapter(adapterClone).harvestCooldown(), 1 days);
2023-01-popcorn-main/test/vault/VaultController.t.sol::505 => 2 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::513 => 1 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::522 => controller.setHarvestCooldown(1 days);
2023-01-popcorn-main/test/vault/VaultController.t.sol::553 => 2 days,
2023-01-popcorn-main/test/vault/VaultController.t.sol::554 => 1 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::576 => controller.setHarvestCooldown(1 days);
2023-01-popcorn-main/test/vault/VaultController.t.sol::680 => controller.setHarvestCooldown(1 days);
2023-01-popcorn-main/test/vault/VaultController.t.sol::717 => 2 days,
2023-01-popcorn-main/test/vault/VaultController.t.sol::718 => 1 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::736 => assertEq(IAdapter(adapterClone).harvestCooldown(), 1 days);
2023-01-popcorn-main/test/vault/VaultController.t.sol::746 => controller.setHarvestCooldown(1 days);
2023-01-popcorn-main/test/vault/VaultController.t.sol::778 => 2 days,
2023-01-popcorn-main/test/vault/VaultController.t.sol::779 => 1 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::847 => 2 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::855 => 1 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::864 => controller.setHarvestCooldown(1 days);
2023-01-popcorn-main/test/vault/VaultController.t.sol::899 => 2 days,
2023-01-popcorn-main/test/vault/VaultController.t.sol::900 => 1 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::941 => controller.setHarvestCooldown(1 days);
2023-01-popcorn-main/test/vault/VaultController.t.sol::953 => assertEq(IAdapter(adapterClone).harvestCooldown(), 1 days);
2023-01-popcorn-main/test/vault/VaultController.t.sol::963 => controller.setHarvestCooldown(1 days);
2023-01-popcorn-main/test/vault/VaultController.t.sol::974 => assertEq(IAdapter(adapterClone).harvestCooldown(), 1 days);
2023-01-popcorn-main/test/vault/VaultController.t.sol::983 => controller.setHarvestCooldown(1 days);
2023-01-popcorn-main/test/vault/VaultController.t.sol::1231 => vm.warp(block.timestamp + 3 days + 1);
2023-01-popcorn-main/test/vault/VaultController.t.sol::1320 => vm.warp(block.timestamp + 3 days + 1);
2023-01-popcorn-main/test/vault/VaultController.t.sol::1379 => 2 days,
2023-01-popcorn-main/test/vault/VaultController.t.sol::1380 => 1 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::1434 => 2 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::1440 => 1 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::1461 => 2 days,
2023-01-popcorn-main/test/vault/VaultController.t.sol::1462 => 1 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::1480 => 2 days,
2023-01-popcorn-main/test/vault/VaultController.t.sol::1481 => 1 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::1496 => 2 days,
2023-01-popcorn-main/test/vault/VaultController.t.sol::1497 => 1 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::1517 => 2 days,
2023-01-popcorn-main/test/vault/VaultController.t.sol::1518 => 1 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::1539 => 2 days,
2023-01-popcorn-main/test/vault/VaultController.t.sol::1540 => 1 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::1562 => 2 days,
2023-01-popcorn-main/test/vault/VaultController.t.sol::1563 => 1 days
2023-01-popcorn-main/test/vault/VaultController.t.sol::2060 => controller.setHarvestCooldown(1 hours);
2023-01-popcorn-main/test/vault/VaultController.t.sol::2061 => assertEq(controller.harvestCooldown(), 1 hours);
2023-01-popcorn-main/test/vault/VaultController.t.sol::2065 => controller.setHarvestCooldown(2 days);
2023-01-popcorn-main/test/vault/VaultController.t.sol::2070 => controller.setHarvestCooldown(1 hours);
2023-01-popcorn-main/test/vault/VaultController.t.sol::2078 => controller.setHarvestCooldown(1 hours);
2023-01-popcorn-main/test/vault/VaultController.t.sol::2081 => assertEq(IAdapter(adapter).harvestCooldown(), 1 hours);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::506 => emit HarvestCooldownChanged(0, 1 hours);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::507 => adapter.setHarvestCooldown(1 hours);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::509 => assertEq(adapter.harvestCooldown(), 1 hours);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::514 => adapter.setHarvestCooldown(1 hours);
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::518 => adapter.setHarvestCooldown(2 days);
```


### [N06] NC-library/interface files should use fixed compiler versions, not floating ones



#### Findings:
```
2023-01-popcorn-main/src/interfaces/IEIP165.sol::2 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/IMultiRewardEscrow.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/IMultiRewardStaking.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IAdapter.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IAdminProxy.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/ICloneFactory.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/ICloneRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IDeploymentController.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IERC4626.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IPermissionRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IStrategy.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/ITemplateRegistry.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IVault.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IVaultController.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IVaultRegistry.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/interfaces/vault/IWithRewards.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/utils/EIP165.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/AdminProxy.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/CloneFactory.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/CloneRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/DeploymentController.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/PermissionRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/TemplateRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/Vault.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/VaultController.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/VaultRegistry.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/adapter/abstracts/OnlyStrategy.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/adapter/abstracts/WithRewards.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/adapter/beefy/BeefyAdapter.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/adapter/beefy/IBeefy.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/adapter/yearn/IYearn.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/src/vault/adapter/yearn/YearnAdapter.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/MultiRewardEscrow.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/utils/EnhancedTest.sol::2 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/utils/Faucet.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/utils/mocks/ClonableWithInitData.sol::1 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/utils/mocks/ClonableWithoutInitData.sol::1 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/utils/mocks/MockAdapter.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/utils/mocks/MockERC20.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/utils/mocks/MockERC4626.sol::3 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/utils/mocks/MockStrategy.sol::1 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/CloneFactory.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/CloneRegistry.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/DeploymentController.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/PermissionRegistry.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/TemplateRegistry.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/Vault.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/VaultController.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/VaultRegistry.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/abstract/ITestConfigStorage.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/abstract/PropertyTest.prop.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/beefy/BeefyAdapter.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/beefy/BeefyTestConfigStorage.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/beefy/BeefyVault.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/yearn/YearnAdapter.t.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/yearn/YearnTestConfigStorage.sol::4 => pragma solidity ^0.8.15;
2023-01-popcorn-main/test/vault/integration/yearn/YearnVault.t.sol::4 => pragma solidity ^0.8.15;
```


### [N07] Interfaces should be moved to separate files

#### Impact
Most of the other interfaces in this project are in their own file in
 the interfaces directory. The interfaces below do not follow this 
pattern
#### Findings:
```
2023-01-popcorn-main/src/vault/adapter/beefy/IBeefy.sol::6 => interface IBeefyVault {
2023-01-popcorn-main/src/vault/adapter/beefy/IBeefy.sol::29 => interface IBeefyBooster {
2023-01-popcorn-main/src/vault/adapter/beefy/IBeefy.sol::51 => interface IBeefyBalanceCheck {
2023-01-popcorn-main/src/vault/adapter/beefy/IBeefy.sol::55 => interface IBeefyStrat {
2023-01-popcorn-main/src/vault/adapter/yearn/IYearn.sol::8 => interface VaultAPI is IERC20 {
2023-01-popcorn-main/src/vault/adapter/yearn/IYearn.sol::32 => interface IYearnRegistry {
2023-01-popcorn-main/test/utils/Faucet.sol::9 => interface Uniswap {
2023-01-popcorn-main/test/utils/Faucet.sol::20 => interface CrvSethPool {
2023-01-popcorn-main/test/utils/Faucet.sol::24 => interface Crv3CryptoPool {
2023-01-popcorn-main/test/utils/Faucet.sol::28 => interface CrvAavePool {
2023-01-popcorn-main/test/utils/Faucet.sol::32 => interface CrvCompPool {
2023-01-popcorn-main/test/utils/Faucet.sol::36 => interface CrvCvxCrvPool {
2023-01-popcorn-main/test/utils/Faucet.sol::40 => interface CrvIbBtcPool {
2023-01-popcorn-main/test/utils/Faucet.sol::44 => interface CrvSBtcPool {
2023-01-popcorn-main/test/utils/Faucet.sol::48 => interface TriPool {
2023-01-popcorn-main/test/utils/Faucet.sol::52 => interface IWETH is IERC20 {
2023-01-popcorn-main/test/utils/Faucet.sol::58 => interface CDai is IERC20 {
2023-01-popcorn-main/test/vault/integration/abstract/ITestConfigStorage.sol::6 => interface ITestConfigStorage {
```



### [N08] SHOWING THE ACTUAL VALUES OF NUMBERS IN NATSPEC COMMENTS MAKES CHECKING AND READING CODE EASIER


#### Findings:
```
2023-01-popcorn-main/src/vault/adapter/beefy/YearnAdapter.sol::25 => uint256 constant DEGRADATION_COEFFICIENT = 10**18;
2023-01-popcorn-main/src/vault/adapter/beefy/BeefyAdapter.sol::31 => uint256 public constant BPS_DENOMINATOR = 10_000;
2023-01-popcorn-main/src/vault/adapter/beefy/Vault.sol::35 => uint256 constant SECONDS_PER_YEAR = 365.25 days;
```







### [N09] MISSING EVENT FOR CRITICAL PARAMETERS INIT AND CHANGE


#### Findings:
```
2023-01-popcorn-main/src/vault/adapter/beefy/MultiRewardEscrow.sol::30-32 =>  constructor(address _owner, address _feeRecipient) Owned(_owner) {
    feeRecipient = _feeRecipient;
  }

2023-01-popcorn-main/src/vault/adapter/beefy/DeploymentController.sol::35-44 =>   constructor(
    address _owner,
    ICloneFactory _cloneFactory,
    ICloneRegistry _cloneRegistry,
    ITemplateRegistry _templateRegistry
  ) Owned(_owner) {
    cloneFactory = _cloneFactory;
    cloneRegistry = _cloneRegistry;
    templateRegistry = _templateRegistry;
  }


2023-01-popcorn-main/src/vault/adapter/beefy/VaultController.sol::53-70 =>   constructor(
    address _owner,
    IAdminProxy _adminProxy,
    IDeploymentController _deploymentController,
    IVaultRegistry _vaultRegistry,
    IPermissionRegistry _permissionRegistry,
    IMultiRewardEscrow _escrow
  ) Owned(_owner) {
    adminProxy = _adminProxy;
    vaultRegistry = _vaultRegistry;
    permissionRegistry = _permissionRegistry;
    escrow = _escrow;

    _setDeploymentController(_deploymentController);

    activeTemplateId[STAKING] = "MultiRewardStaking";
    activeTemplateId[VAULT] = "V1";
  }
```
