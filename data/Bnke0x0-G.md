
### [G01] State variables only set in the constructor should be declared `immutable`

#### Impact
Avoids a Gusset (20000 gas)
#### Findings:
```
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::191 => address public feeRecipient;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::31 => string private _name;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::32 => string private _symbol;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::33 => uint8 private _decimals;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::157 => IMultiRewardEscrow public escrow;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::208 => IERC20[] public rewardTokens;
2023-01-popcorn-main/src/vault/CloneRegistry.sol::31 => address[] public allClones;
2023-01-popcorn-main/src/vault/DeploymentController.sol::23 => ICloneFactory public cloneFactory;
2023-01-popcorn-main/src/vault/DeploymentController.sol::24 => ICloneRegistry public cloneRegistry;
2023-01-popcorn-main/src/vault/DeploymentController.sol::25 => ITemplateRegistry public templateRegistry;
2023-01-popcorn-main/src/vault/TemplateRegistry.sol::36 => bytes32[] public templateCategories;
2023-01-popcorn-main/src/vault/Vault.sol::37 => IERC20 public asset;
2023-01-popcorn-main/src/vault/Vault.sol::40 => bytes32 public contractName;
2023-01-popcorn-main/src/vault/Vault.sol::466 => uint256 public highWaterMark = 1e18;
2023-01-popcorn-main/src/vault/Vault.sol::467 => uint256 public assetsCheckpoint;
2023-01-popcorn-main/src/vault/Vault.sol::468 => uint256 public feesUpdatedAt;
2023-01-popcorn-main/src/vault/Vault.sol::505 => VaultFees public fees;
2023-01-popcorn-main/src/vault/Vault.sol::507 => VaultFees public proposedFees;
2023-01-popcorn-main/src/vault/Vault.sol::508 => uint256 public proposedFeeTime;
2023-01-popcorn-main/src/vault/Vault.sol::510 => address public feeRecipient;
2023-01-popcorn-main/src/vault/Vault.sol::565 => IERC4626 public adapter;
2023-01-popcorn-main/src/vault/Vault.sol::566 => IERC4626 public proposedAdapter;
2023-01-popcorn-main/src/vault/Vault.sol::567 => uint256 public proposedAdapterTime;
2023-01-popcorn-main/src/vault/Vault.sol::619 => uint256 public quitPeriod = 3 days;
2023-01-popcorn-main/src/vault/VaultController.sol::387 => IVaultRegistry public vaultRegistry;
2023-01-popcorn-main/src/vault/VaultController.sol::535 => IMultiRewardEscrow public escrow;
2023-01-popcorn-main/src/vault/VaultController.sol::717 => IAdminProxy public adminProxy;
2023-01-popcorn-main/src/vault/VaultController.sol::739 => uint256 public performanceFee;
2023-01-popcorn-main/src/vault/VaultController.sol::779 => uint256 public harvestCooldown;
2023-01-popcorn-main/src/vault/VaultController.sol::819 => IDeploymentController public deploymentController;
2023-01-popcorn-main/src/vault/VaultController.sol::820 => ICloneRegistry public cloneRegistry;
2023-01-popcorn-main/src/vault/VaultController.sol::821 => ITemplateRegistry public templateRegistry;
2023-01-popcorn-main/src/vault/VaultController.sol::822 => IPermissionRegistry public permissionRegistry;
2023-01-popcorn-main/src/vault/VaultRegistry.sol::34 => address[] public allVaults;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::427 => IStrategy public strategy;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::428 => bytes public strategyConfig;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::429 => uint256 public lastHarvest;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::489 => uint256 public harvestCooldown;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::513 => uint256 public performanceFee;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::514 => uint256 public highWaterMark;
2023-01-popcorn-main/src/vault/adapter/beefy/BeefyAdapter.sol::27 => IBeefyVault public beefyVault;
2023-01-popcorn-main/src/vault/adapter/beefy/BeefyAdapter.sol::28 => IBeefyBooster public beefyBooster;
2023-01-popcorn-main/src/vault/adapter/beefy/BeefyAdapter.sol::29 => IBeefyBalanceCheck public beefyBalanceCheck;
2023-01-popcorn-main/src/vault/adapter/beefy/BeefyAdapter.sol::31 => uint256 public constant BPS_DENOMINATOR = 10_000;
2023-01-popcorn-main/src/vault/adapter/yearn/YearnAdapter.sol::24 => VaultAPI public yVault;
2023-01-popcorn-main/test/utils/Faucet.sol::66 => Uniswap public uniswap;
2023-01-popcorn-main/test/utils/mocks/ClonableWithInitData.sol::6 => uint256 public val;
2023-01-popcorn-main/test/utils/mocks/ClonableWithoutInitData.sol::8 => bool public initDone;
2023-01-popcorn-main/test/utils/mocks/MockAdapter.sol::22 => uint256 public initValue;
```



### [G02] State variables can be packed into fewer storage slots

#### Impact
If variables occupying the same slot are both written the same 
function or by the constructor avoids a separate Gsset (20000 gas). 

#### Findings:
```
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::191 => address public feeRecipient;
2023-01-popcorn-main/src/vault/Vault.sol::510 => address public feeRecipient;

```

### [G03] `++i/i++` should be `unchecked{++i}`/`unchecked{++i}` when it is not possible for them to overflow, as is the case when used in `for` and `while` loops


#### Findings:
```
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::53 => for (uint256 i = 0; i < escrowIds.length; i++) {
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::155 => for (uint256 i = 0; i < escrowIds.length; i++) {
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::210 => for (uint256 i = 0; i < tokens.length; i++) {
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::171 => for (uint8 i; i < _rewardTokens.length; i++) {
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::373 => for (uint8 i; i < _rewardTokens.length; i++) {
2023-01-popcorn-main/src/vault/PermissionRegistry.sol::42 => for (uint256 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::319 => for (uint8 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::337 => for (uint8 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::357 => for (uint8 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::374 => for (uint8 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::437 => for (uint256 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::494 => for (uint256 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::523 => for (uint256 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::564 => for (uint256 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::587 => for (uint256 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::607 => for (uint256 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::620 => for (uint256 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::633 => for (uint256 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::646 => for (uint256 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::766 => for (uint256 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::806 => for (uint256 i = 0; i < len; i++) {
2023-01-popcorn-main/test/vault/VaultRegistry.t.sol::30 => for (uint256 i; i < 8; ++i) {
2023-01-popcorn-main/test/vault/VaultRegistry.t.sol::65 => for (uint256 i; i < 8; ++i) {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::268 => for (uint8 i; i < len; i++) {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::284 => for (uint8 i; i < len; i++) {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::300 => for (uint8 i; i < len; i++) {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::324 => for (uint8 i; i < len; i++) {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::155 => for (uint256 i; i < len; i++) {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::159 => for (uint256 i; i < 3; ++i) {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::166 => for (uint256 i; i < 2; ++i) {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::181 => for (uint256 i; i < len; i++) {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::210 => for (uint256 i; i < len; i++) {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::214 => for (uint256 i; i < 3; ++i) {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::221 => for (uint256 i; i < 2; ++i) {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::236 => for (uint256 i; i < len; i++) {
```



### [G04] Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

#### Impact
> When using elements that are smaller than 32 bytes, your 
contract’s gas usage may be higher. This is because the EVM operates on 
32 bytes at a time. Therefore, if the element is smaller than that, the 
EVM must use more operations in order to reduce the size of the element 
from 32 bytes to the desired size.
> 

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html)
Use a larger size then downcast where needed
#### Findings:
```
2023-01-popcorn-main/src/interfaces/IMultiRewardEscrow.sol::11 => uint32 start;
2023-01-popcorn-main/src/interfaces/IMultiRewardEscrow.sol::13 => uint32 end;
2023-01-popcorn-main/src/interfaces/IMultiRewardEscrow.sol::15 => uint32 lastUpdateTime;
2023-01-popcorn-main/src/interfaces/IMultiRewardEscrow.sol::36 => uint32 duration,
2023-01-popcorn-main/src/interfaces/IMultiRewardEscrow.sol::37 => uint32 offset
2023-01-popcorn-main/src/interfaces/IMultiRewardStaking.sol::16 => uint64 ONE;
2023-01-popcorn-main/src/interfaces/IMultiRewardStaking.sol::18 => uint160 rewardsPerSecond;
2023-01-popcorn-main/src/interfaces/IMultiRewardStaking.sol::21 => uint32 rewardsEndTimestamp;
2023-01-popcorn-main/src/interfaces/IMultiRewardStaking.sol::23 => uint224 index;
2023-01-popcorn-main/src/interfaces/IMultiRewardStaking.sol::25 => uint32 lastUpdatedTimestamp;
2023-01-popcorn-main/src/interfaces/IMultiRewardStaking.sol::30 => uint192 escrowPercentage;
2023-01-popcorn-main/src/interfaces/IMultiRewardStaking.sol::32 => uint32 escrowDuration;
2023-01-popcorn-main/src/interfaces/IMultiRewardStaking.sol::34 => uint32 offset;
2023-01-popcorn-main/src/interfaces/IMultiRewardStaking.sol::40 => uint160 rewardsPerSecond,
2023-01-popcorn-main/src/interfaces/IMultiRewardStaking.sol::43 => uint192 escrowPercentage,
2023-01-popcorn-main/src/interfaces/IMultiRewardStaking.sol::44 => uint32 escrowDuration,
2023-01-popcorn-main/src/interfaces/IMultiRewardStaking.sol::45 => uint32 offset
2023-01-popcorn-main/src/interfaces/vault/IVault.sol::9 => uint64 deposit;
2023-01-popcorn-main/src/interfaces/vault/IVault.sol::10 => uint64 withdrawal;
2023-01-popcorn-main/src/interfaces/vault/IVault.sol::11 => uint64 management;
2023-01-popcorn-main/src/interfaces/vault/IVault.sol::12 => uint64 performance;
2023-01-popcorn-main/src/interfaces/vault/IVaultController.sol::60 => uint160[] memory rewardsSpeeds
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::92 => uint32 duration,
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::93 => uint32 offset
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::114 => uint32 start = block.timestamp.safeCastTo32() + offset;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::33 => uint8 private _decimals;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::67 => function decimals() public view override(ERC20Upgradeable, IERC20Metadata) returns (uint8) {
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::171 => for (uint8 i; i < _rewardTokens.length; i++) {
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::220 => event RewardInfoUpdate(IERC20 rewardToken, uint160 rewardsPerSecond, uint32 rewardsEndTimestamp);
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::245 => uint160 rewardsPerSecond,
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::248 => uint192 escrowPercentage,
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::249 => uint32 escrowDuration,
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::250 => uint32 offset
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::274 => uint64 ONE = (10**IERC20Metadata(address(rewardToken)).decimals()).safeCastTo64();
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::275 => uint32 rewardsEndTimestamp = rewardsPerSecond == 0
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::296 => function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::307 => uint32 prevEndTime = rewards.rewardsEndTimestamp;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::308 => uint32 rewardsEndTimestamp = _calcRewardsEnd(
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::340 => uint32 rewardsEndTimestamp = rewards.rewardsEndTimestamp;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::352 => uint32 rewardsEndTimestamp,
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::353 => uint160 rewardsPerSecond,
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::355 => ) internal returns (uint32) {
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::373 => for (uint8 i; i < _rewardTokens.length; i++) {
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::404 => uint224 deltaIndex;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::450 => uint8 v,
2023-01-popcorn-main/src/vault/Vault.sol::38 => uint8 internal _decimals;
2023-01-popcorn-main/src/vault/Vault.sol::100 => function decimals() public view override returns (uint8) {
2023-01-popcorn-main/src/vault/Vault.sol::669 => uint8 v,
2023-01-popcorn-main/src/vault/VaultController.sol::314 => uint8 len = uint8(vaults.length);
2023-01-popcorn-main/src/vault/VaultController.sol::319 => for (uint8 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::336 => uint8 len = uint8(vaults.length);
2023-01-popcorn-main/src/vault/VaultController.sol::337 => for (uint8 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::353 => uint8 len = uint8(vaults.length);
2023-01-popcorn-main/src/vault/VaultController.sol::357 => for (uint8 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::373 => uint8 len = uint8(vaults.length);
2023-01-popcorn-main/src/vault/VaultController.sol::374 => for (uint8 i = 0; i < len; i++) {
2023-01-popcorn-main/src/vault/VaultController.sol::436 => uint8 len = uint8(vaults.length);
2023-01-popcorn-main/src/vault/VaultController.sol::440 => uint160 rewardsPerSecond,
2023-01-popcorn-main/src/vault/VaultController.sol::443 => uint224 escrowDuration,
2023-01-popcorn-main/src/vault/VaultController.sol::444 => uint24 escrowPercentage,
2023-01-popcorn-main/src/vault/VaultController.sol::486 => uint160[] calldata rewardsSpeeds
2023-01-popcorn-main/src/vault/VaultController.sol::488 => uint8 len = uint8(vaults.length);
2023-01-popcorn-main/src/vault/VaultController.sol::517 => uint8 len = uint8(vaults.length);
2023-01-popcorn-main/src/vault/VaultController.sol::563 => uint8 len = uint8(templateCategories.length);
2023-01-popcorn-main/src/vault/VaultController.sol::583 => uint8 len = uint8(templateCategories.length);
2023-01-popcorn-main/src/vault/VaultController.sol::606 => uint8 len = uint8(vaults.length);
2023-01-popcorn-main/src/vault/VaultController.sol::619 => uint8 len = uint8(vaults.length);
2023-01-popcorn-main/src/vault/VaultController.sol::632 => uint8 len = uint8(vaults.length);
2023-01-popcorn-main/src/vault/VaultController.sol::645 => uint8 len = uint8(vaults.length);
2023-01-popcorn-main/src/vault/VaultController.sol::765 => uint8 len = uint8(adapters.length);
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::38 => uint8 internal _decimals;
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::93 => returns (uint8)
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::637 => uint8 v,
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::31 => event RewardInfoUpdate(IERC20 rewardsToken, uint160 rewardsPerSecond, uint32 rewardsEndTimestamp);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::82 => function test__single_deposit_withdraw(uint128 amount) public {
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::155 => function test__single_mint_redeem(uint128 amount) public {
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::224 => (uint8 v, bytes32 r, bytes32 s) = vm.sign(
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::259 => (, , , uint224 index, uint32 lastUpdatedTimestamp) = staking.rewardInfos(iRewardToken1);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::347 => (, , , uint224 indexReward2, uint32 lastUpdatedTimestampReward2) = staking.rewardInfos(iRewardToken2);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::507 => (, , , uint224 index, uint32 lastUpdatedTimestamp) = staking.rewardInfos(iRewardToken1);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::543 => (, , , uint224 index, uint32 lastUpdatedTimestamp) = staking.rewardInfos(iRewardToken1);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::626 => (uint192 escrowPercentage, uint32 escrowDuration, uint32 offset) = staking.escrowInfos(iRewardToken1);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::632 => uint64 ONE,
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::633 => uint160 rewardsPerSecond,
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::634 => uint32 rewardsEndTimestamp,
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::635 => uint224 index,
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::636 => uint32 lastUpdatedTimestamp
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::657 => uint64 ONE,
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::658 => uint160 rewardsPerSecond,
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::659 => uint32 rewardsEndTimestamp,
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::660 => uint224 index,
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::661 => uint32 lastUpdatedTimestamp
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::682 => (, , uint32 rewardsEndTimestamp, , ) = staking.rewardInfos(iRewardToken1);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::750 => (, , , uint224 index, uint32 lastUpdatedTimestamp) = staking.rewardInfos(iRewardToken1);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::796 => (, , uint32 oldRewardsEndTimestamp, , ) = staking.rewardInfos(iRewardToken1);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::804 => (, , uint32 rewardsEndTimestamp, , ) = staking.rewardInfos(iRewardToken1);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::818 => (, , uint32 oldRewardsEndTimestamp, , ) = staking.rewardInfos(iRewardToken1);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::826 => (, , uint32 rewardsEndTimestamp, , ) = staking.rewardInfos(iRewardToken1);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::843 => (, , uint32 oldRewardsEndTimestamp, , ) = staking.rewardInfos(iRewardToken1);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::848 => (, , uint32 rewardsEndTimestamp, , ) = staking.rewardInfos(iRewardToken1);
2023-01-popcorn-main/test/utils/EnhancedTest.sol::11 => emit log("Error: a >=~ b not satisfied [uint]");
2023-01-popcorn-main/test/utils/EnhancedTest.sol::12 => emit log_named_uint("   Value a", a);
2023-01-popcorn-main/test/utils/EnhancedTest.sol::13 => emit log_named_uint("   Value b", b);
2023-01-popcorn-main/test/utils/EnhancedTest.sol::14 => emit log_named_uint(" Max Delta", maxDelta);
2023-01-popcorn-main/test/utils/EnhancedTest.sol::15 => emit log_named_uint("     Delta", dt);
2023-01-popcorn-main/test/utils/EnhancedTest.sol::26 => emit log("Error: a >=~ b not satisfied [uint]");
2023-01-popcorn-main/test/utils/EnhancedTest.sol::27 => emit log_named_uint("   Value a", a);
2023-01-popcorn-main/test/utils/EnhancedTest.sol::28 => emit log_named_uint("   Value b", b);
2023-01-popcorn-main/test/utils/EnhancedTest.sol::29 => emit log_named_uint(" Max Delta", maxDelta);
2023-01-popcorn-main/test/utils/EnhancedTest.sol::30 => emit log_named_uint("     Delta", dt);
2023-01-popcorn-main/test/utils/EnhancedTest.sol::40 => emit log("Error: a <=~ b not satisfied [uint]");
2023-01-popcorn-main/test/utils/EnhancedTest.sol::41 => emit log_named_uint("   Value a", a);
2023-01-popcorn-main/test/utils/EnhancedTest.sol::42 => emit log_named_uint("   Value b", b);
2023-01-popcorn-main/test/utils/EnhancedTest.sol::43 => emit log_named_uint(" Max Delta", maxDelta);
2023-01-popcorn-main/test/utils/EnhancedTest.sol::44 => emit log_named_uint("     Delta", dt);
2023-01-popcorn-main/test/utils/EnhancedTest.sol::55 => emit log("Error: a <=~ b not satisfied [uint]");
2023-01-popcorn-main/test/utils/EnhancedTest.sol::56 => emit log_named_uint("   Value a", a);
2023-01-popcorn-main/test/utils/EnhancedTest.sol::57 => emit log_named_uint("   Value b", b);
2023-01-popcorn-main/test/utils/EnhancedTest.sol::58 => emit log_named_uint(" Max Delta", maxDelta);
2023-01-popcorn-main/test/utils/EnhancedTest.sol::59 => emit log_named_uint("     Delta", dt);
2023-01-popcorn-main/test/utils/mocks/MockERC20.sol::8 => uint8 internal _decimals;
2023-01-popcorn-main/test/utils/mocks/MockERC20.sol::13 => uint8 decimals_
2023-01-popcorn-main/test/utils/mocks/MockERC20.sol::18 => function decimals() public view override returns (uint8) {
2023-01-popcorn-main/test/utils/mocks/MockERC4626.sol::18 => uint8 internal _decimals;
2023-01-popcorn-main/test/vault/Vault.t.sol::65 => uint64 depositFee,
2023-01-popcorn-main/test/vault/Vault.t.sol::66 => uint64 withdrawalFee,
2023-01-popcorn-main/test/vault/Vault.t.sol::67 => uint64 managementFee,
2023-01-popcorn-main/test/vault/Vault.t.sol::68 => uint64 performanceFee
2023-01-popcorn-main/test/vault/Vault.t.sol::183 => function test__deposit_withdraw(uint128 amount) public {
2023-01-popcorn-main/test/vault/Vault.t.sol::259 => function test__mint_redeem(uint128 amount) public {
2023-01-popcorn-main/test/vault/Vault.t.sol::593 => function test__previewDeposit_previewMint_takes_fees_into_account(uint8 fuzzAmount) public {
2023-01-popcorn-main/test/vault/Vault.t.sol::611 => function test__previewWithdraw_previewRedeem_takes_fees_into_account(uint8 fuzzAmount) public {
2023-01-popcorn-main/test/vault/Vault.t.sol::646 => function test__managementFee(uint128 timeframe) public {
2023-01-popcorn-main/test/vault/Vault.t.sol::676 => function test__performanceFee(uint128 amount) public {
2023-01-popcorn-main/test/vault/Vault.t.sol::992 => (uint8 v, bytes32 r, bytes32 s) = vm.sign(
2023-01-popcorn-main/test/vault/VaultController.t.sol::119 => uint32 duration,
2023-01-popcorn-main/test/vault/VaultController.t.sol::120 => uint32 offset
2023-01-popcorn-main/test/vault/VaultController.t.sol::133 => uint160 rewardsPerSecond,
2023-01-popcorn-main/test/vault/VaultController.t.sol::134 => uint32 rewardsEndTimestamp
2023-01-popcorn-main/test/vault/VaultController.t.sol::1577 => uint160[] memory rewardSpeeds = new uint160[](1);
2023-01-popcorn-main/test/vault/VaultController.t.sol::1604 => uint160[] memory rewardSpeeds = new uint160[](2);
2023-01-popcorn-main/test/vault/VaultController.t.sol::1616 => uint160[] memory rewardSpeeds = new uint160[](1);
2023-01-popcorn-main/test/vault/VaultRegistry.t.sol::31 => swapTokenAddresses[i] = address(uint160(i));
2023-01-popcorn-main/test/vault/VaultRegistry.t.sol::66 => assertEq(savedVault.swapTokenAddresses[i], address(uint160(i)));
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::134 => emit Initialized(uint8(1));
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::219 => function test__previewDeposit(uint8 fuzzAmount) public virtual {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::250 => function test__previewRedeem(uint8 fuzzAmount) public virtual {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::267 => uint8 len = uint8(testConfigStorage.getTestConfigLength());
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::268 => for (uint8 i; i < len; i++) {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::281 => function test__mint(uint8 fuzzAmount) public virtual {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::283 => uint8 len = uint8(testConfigStorage.getTestConfigLength());
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::284 => for (uint8 i; i < len; i++) {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::297 => function test__withdraw(uint8 fuzzAmount) public virtual {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::299 => uint8 len = uint8(testConfigStorage.getTestConfigLength());
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::300 => for (uint8 i; i < len; i++) {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::323 => uint8 len = uint8(testConfigStorage.getTestConfigLength());
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::324 => for (uint8 i; i < len; i++) {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::148 => function test__deposit_withdrawl_pps_stays_constant(uint80 fuzzAmount) public {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::176 => function test__deposit_withdrawl_preview_resembles_actual(uint80 fuzzAmount) public {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::203 => function test__mint_redeem_pps_stays_constant(uint80 fuzzAmount) public {
2023-01-popcorn-main/test/vault/integration/abstract/AbstractVaultIntegrationTest.sol::231 => function test__mint_redeem_preview_resembles_actual(uint80 fuzzAmount) public {
```


### [G05] Duplicated `require()`/`revert()` checks should be refactored to a modifier or function



#### Findings:
```
2023-01-popcorn-main/src/vault/Vault.sol::542 => revert NotPassedQuitPeriod(quitPeriod);
2023-01-popcorn-main/src/vault/Vault.sol::596 => revert NotPassedQuitPeriod(quitPeriod);
```



### [G06] Bitshift for divide by 2

#### Impact
When multiplying or dividing by a power of two, it is cheaper to bitshift than to use standard math operations. 
#### Findings:
```
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::67 => maxShares = maxAssets / 2;
```









### [G07] Amounts should be checked for 0 before calling a transfer

#### Impact
Checking non-zero transfer values can avoid an expensive external call and save gas.

While this is done in some places, it’s not consistently done in the solution.
#### Findings:
```
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::111 => token.safeTransfer(feeRecipient, fee);
2023-01-popcorn-main/src/utils/MultiRewardEscrow.sol::165 => escrow.token.safeTransfer(escrow.account, claimable);
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::182 => _rewardTokens[i].transfer(user, rewardAmount);
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::200 => rewardToken.safeTransfer(user, payout);
2023-01-popcorn-main/src/vault/VaultController.sol::526 => rewardTokens[i].transferFrom(msg.sender, address(this), amounts[i]);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::428 => staking.transfer(bob, 1 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::445 => staking.transferFrom(bob, alice, 1 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::464 => staking.transferFrom(alice, address(this), 1 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::676 => rewardToken1.transfer(address(staking), 10 ether);
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::841 => rewardToken1.transfer(address(staking), 10 ether);
2023-01-popcorn-main/test/utils/Faucet.sol::119 => crv3CryptoLP.transfer(recipient, crv3CryptoLP.balanceOf(address(this)));
2023-01-popcorn-main/test/utils/Faucet.sol::125 => crvAaveLP.transfer(recipient, crvAaveLP.balanceOf(address(this)));
2023-01-popcorn-main/test/utils/Faucet.sol::133 => crvCompLP.transfer(recipient, crvCompLP.balanceOf(address(this)));
2023-01-popcorn-main/test/utils/Faucet.sol::139 => crvCvxCrvLP.transfer(recipient, crvCvxCrvLP.balanceOf(address(this)));
2023-01-popcorn-main/test/utils/Faucet.sol::146 => crvIbBtcLP.transfer(recipient, crvIbBtcLP.balanceOf(address(this)));
2023-01-popcorn-main/test/utils/Faucet.sol::151 => crvSethLP.transfer(recipient, crvSethLP.balanceOf(address(this)));
2023-01-popcorn-main/test/utils/Faucet.sol::157 => threeCrv.transfer(recipient, threeCrv.balanceOf(address(this)));
2023-01-popcorn-main/test/utils/mocks/MockAdapter.sol::15 => asset.transfer(msg.sender, amount);
2023-01-popcorn-main/test/utils/mocks/MockERC4626.sol::104 => asset.safeTransfer(receiver, assets);
2023-01-popcorn-main/test/utils/mocks/MockERC4626.sol::123 => asset.safeTransfer(receiver, assets);
```






### [G08] Multiple `address` mappings can be combined into a single `mapping` of an `address` to a `struct`, where appropriate

#### Impact
Saves a storage slot for the mapping. Depending on the circumstances 
and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. 
Reads and subsequent writes can also be cheaper when a function requires
 both values and they both fit in the same storage slot
#### Findings:
```
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::211 => mapping(IERC20 => RewardInfo) public rewardInfos;
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::216 => mapping(address => mapping(IERC20 => uint256)) public userIndex;
2023-01-popcorn-main/src/vault/TemplateRegistry.sol::35 => mapping(bytes32 => bool) public templateCategoryExists;
```



### [G09] Using `bools` for storage incurs overhead


#### Findings:
```
2023-01-popcorn-main/src/vault/CloneRegistry.sol::28 => mapping(address => bool) public cloneExists;
2023-01-popcorn-main/src/vault/TemplateRegistry.sol::33 => mapping(bytes32 => bool) public templateExists;
2023-01-popcorn-main/src/vault/TemplateRegistry.sol::35 => mapping(bytes32 => bool) public templateCategoryExists;
2023-01-popcorn-main/test/utils/mocks/ClonableWithoutInitData.sol::8 => bool public initDone;
```



### [G10] Empty blocks should be removed or emit something

#### Impact
The code should be refactored such that they no longer exist, or the 
block should do something useful, such as emitting an event or 
reverting. If the contract is meant to be extended, the contract should 
be abstract and the function signatures be added without 
any default implementation. If the block is an empty if-statement block 
to avoid doing subsequent checks in the else-if/else conditions, the 
else-if/else conditions should be nested under the negation of the 
if-statement, because they involve different classes of checks, which 
may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}})
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


### [G11] `keccak256()` should only need to be called on a specific string literal once

#### Impact
It should be saved to an immutable variable, and the variable used 
instead. If the hash is being used as a part of a function selector, the
 cast to bytes4 should also only be done once
#### Findings:
```
2023-01-popcorn-main/test/vault/CloneFactory.t.sol::49 => abi.encodeWithSelector(bytes4(keccak256("initialize()")), "")
2023-01-popcorn-main/test/vault/CloneFactory.t.sol::56 => abi.encodeWithSelector(bytes4(keccak256("initialize()")), "")
```



### [G12] `abi.encode()` is less efficient than abi.encodePacked()



#### Findings:
```
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::465 => abi.encode(
2023-01-popcorn-main/src/utils/MultiRewardStaking.sol::494 => abi.encode(
2023-01-popcorn-main/src/vault/Vault.sol::684 => abi.encode(
2023-01-popcorn-main/src/vault/Vault.sol::719 => abi.encode(
2023-01-popcorn-main/src/vault/VaultController.sol::219 => abi.encode(asset, address(adminProxy), IStrategy(strategy), harvestCooldown, requiredSigs, strategyData.data),
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::652 => abi.encode(
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::687 => abi.encode(
2023-01-popcorn-main/test/MultiRewardStaking.t.sol::230 => keccak256(abi.encode(PERMIT_TYPEHASH, owner, address(0xCAFE), 1e18, 0, block.timestamp))
2023-01-popcorn-main/test/vault/Vault.t.sol::998 => keccak256(abi.encode(PERMIT_TYPEHASH, owner, address(0xCAFE), 1e18, 0, block.timestamp))
2023-01-popcorn-main/test/vault/VaultController.t.sol::276 => data: abi.encode(uint256(100))
2023-01-popcorn-main/test/vault/VaultController.t.sol::303 => data: abi.encode(uint256(100))
2023-01-popcorn-main/test/vault/VaultController.t.sol::307 => abi.encode(
2023-01-popcorn-main/test/vault/VaultController.t.sol::397 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::400 => abi.encode(
2023-01-popcorn-main/test/vault/VaultController.t.sol::544 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::547 => abi.encode(
2023-01-popcorn-main/test/vault/VaultController.t.sol::598 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::689 => DeploymentArgs({id: templateId, data: abi.encode(uint256(300))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::711 => abi.encode(
2023-01-popcorn-main/test/vault/VaultController.t.sol::769 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::772 => abi.encode(
2023-01-popcorn-main/test/vault/VaultController.t.sol::890 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::893 => abi.encode(
2023-01-popcorn-main/test/vault/VaultController.t.sol::947 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::968 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::991 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::1008 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::1021 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::1032 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::1043 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::1050 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::1062 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::1075 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::1146 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::1223 => DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
2023-01-popcorn-main/test/vault/VaultController.t.sol::1373 => rewardsData[0] = abi.encode(
2023-01-popcorn-main/test/vault/VaultController.t.sol::1455 => rewardsData[0] = abi.encode(
2023-01-popcorn-main/test/vault/VaultController.t.sol::1474 => rewardsData[0] = abi.encode(
2023-01-popcorn-main/test/vault/VaultController.t.sol::1490 => rewardsData[0] = abi.encode(
2023-01-popcorn-main/test/vault/VaultController.t.sol::1511 => rewardsData[0] = abi.encode(
2023-01-popcorn-main/test/vault/VaultController.t.sol::1533 => rewardsData[0] = abi.encode(
2023-01-popcorn-main/test/vault/VaultController.t.sol::1556 => rewardsData[0] = abi.encode(
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::136 => abi.encode(asset, address(this), strategy, 0, sigs, ""),
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::565 => keccak256(abi.encode(PERMIT_TYPEHASH, owner, address(0xCAFE), 1e18, 0, block.timestamp))
2023-01-popcorn-main/test/vault/integration/beefy/BeefyAdapter.t.sol::54 => adapter.initialize(abi.encode(asset, address(this), strategy, 0, sigs, ""), externalRegistry, testConfig);
2023-01-popcorn-main/test/vault/integration/beefy/BeefyAdapter.t.sol::133 => abi.encode(asset, address(this), strategy, 0, sigs, ""),
2023-01-popcorn-main/test/vault/integration/beefy/BeefyAdapter.t.sol::135 => abi.encode(_beefyVault, _beefyBooster)
2023-01-popcorn-main/test/vault/integration/beefy/BeefyAdapter.t.sol::149 => abi.encode(asset, address(this), strategy, 0, sigs, ""),
2023-01-popcorn-main/test/vault/integration/beefy/BeefyAdapter.t.sol::151 => abi.encode(address(0x3af3563Ba5C68EB7DCbAdd2dF0FcE4CC9818e75c), _beefyBooster)
2023-01-popcorn-main/test/vault/integration/beefy/BeefyAdapter.t.sol::162 => abi.encode(asset, address(this), strategy, 0, sigs, ""),
2023-01-popcorn-main/test/vault/integration/beefy/BeefyAdapter.t.sol::164 => abi.encode(_beefyVault, address(0xBb77dDe3101B8f9B71755ABe2F69b64e79AE4A41))
2023-01-popcorn-main/test/vault/integration/beefy/BeefyTestConfigStorage.sol::29 => return abi.encode(testConfigs[i].beefyVault, testConfigs[i].beefyBooster);
2023-01-popcorn-main/test/vault/integration/beefy/BeefyVault.t.sol::52 => abi.encode(IERC20(_asset), address(this), strategy, 0, sigs, ""),
2023-01-popcorn-main/test/vault/integration/yearn/YearnAdapter.t.sol::43 => adapter.initialize(abi.encode(asset, address(this), address(0), 0, sigs, ""), externalRegistry, "");
2023-01-popcorn-main/test/vault/integration/yearn/YearnTestConfigStorage.sol::24 => return abi.encode(testConfigs[i].asset);
2023-01-popcorn-main/test/vault/integration/yearn/YearnVault.t.sol::36 => adapter.initialize(abi.encode(_asset, address(this), address(0), 0, sigs, ""), yearnRegistry, "");
```



### [G13] Multiplication/division by two should use bit shifting

#### Impact
<x> * 2 is equivalent to <x> << 1 and <x> / 2 is the same as <x> >> 1. The MUL and DIV opcodes cost 5 gas, whereas SHL and SHR only cost 3 gas
#### Findings:
```
2023-01-popcorn-main/test/vault/integration/abstract/AbstractAdapterTest.sol::67 => maxShares = maxAssets / 2;
```



### [G14] Optimize names to save gas

#### Impact
public/external function names and public member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9)
 link for an example of how it works. Below are the interfaces/abstract 
contracts that can be optimized so that the most frequently-called 
functions use the least amount of gas possible during method lookup. 
Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)
#### Findings:
```
2023-01-popcorn-main/src/vault/adapter/abstracts/AdapterBase.sol::27 => abstract contract AdapterBase is
```


