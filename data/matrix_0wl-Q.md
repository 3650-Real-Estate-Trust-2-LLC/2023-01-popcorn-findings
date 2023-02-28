# Summary

## Low Risk and Non-Critical Issues

## Non-Critical Issues

|       | Issue                                                                      |
| ----- | :------------------------------------------------------------------------- |
| NC-1  | NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING |
| NC-2  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                                       |
| NC-3  | USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)                       |
| NC-4  | SIGNATURE MALLEABILITY OF EVM’S ECRECOVER()                                |
| NC-5  | LACK OF EVENT EMISSION AFTER CRITICAL `INITIALIZE()` FUNCTIONS             |
| NC-6  | ALLOWS MALLEABLE SECP256K1 SIGNATURES                                      |
| NC-7  | NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS                          |
| NC-8  | INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS                              |
| NC-9  | NO SAME VALUE INPUT CONTROL                                                |
| NC-10 | Omissions in events                                                        |
| NC-11 | USE A MORE RECENT VERSION OF SOLIDITY                                      |
| NC-12 | For functions, follow solidity standard naming conventions                 |
| NC-13 | Lines are too long                                                         |
| NC-14 | NOT VERIFIED INPUT                                                         |

### [NC-1] NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING

#### **Proof Of Concept**

```solidity
File: src/interfaces/vault/IERC4626.sol

20:   function asset() external view returns (address assetTokenAddress);

22:   function totalAssets() external view returns (uint256 totalManagedAssets);

26:   function convertToShares(uint256 assets) external view returns (uint256 shares);

28:   function convertToAssets(uint256 shares) external view returns (uint256 assets);

32:   function maxDeposit(address receiver) external view returns (uint256 maxAssets);

34:   function previewDeposit(uint256 assets) external view returns (uint256 shares);

36:   function deposit(uint256 assets, address receiver) external returns (uint256 shares);

40:   function maxMint(address receiver) external view returns (uint256 maxShares);

42:   function previewMint(uint256 shares) external view returns (uint256 assets);

44:   function mint(uint256 shares, address receiver) external returns (uint256 assets);

48:   function maxWithdraw(address owner) external view returns (uint256 maxAssets);

50:   function previewWithdraw(uint256 assets) external view returns (uint256 shares);

52:   function withdraw(uint256 assets, address receiver, address owner) external returns (uint256 shares);

56:   function maxRedeem(address owner) external view returns (uint256 maxShares);

58:   function previewRedeem(uint256 shares) external view returns (uint256 assets);

60:   function redeem(uint256 shares, address receiver, address owner) external returns (uint256 assets);

```

```solidity
File: src/vault/AdminProxy.sol

22:     returns (bool success, bytes memory returndata)

```

```solidity
File: src/vault/CloneFactory.sol

39:   function deploy(Template calldata template, bytes calldata data) external onlyOwner returns (address clone) {

```

```solidity
File: src/vault/DeploymentController.sol

103:   ) external onlyOwner returns (address clone) {

```

```solidity
File: src/vault/VaultController.sol

97:   ) external canCreate returns (address vault) {

122:     returns (address vault)

191:   ) external canCreate returns (address adapter) {

229:   ) internal returns (address adapter) {

244:     returns (bytes memory)

258:     returns (address strategy)

286:     returns (address staking)

667:   function _verifyCreatorOrOwner(address vault) internal returns (VaultMetadata memory metadata) {

673:   function _verifyCreator(address vault) internal view returns (VaultMetadata memory metadata) {

```

#### Recommended Mitigation Steps:

Consider adopting a consistent approach to return values throughout the codebase by removing all named return variables, explicitly declaring them as local variables, and adding the necessary return statements where appropriate. This would improve both the explicitness and readability of the code, and it may also help reduce regressions during future code refactors.

### [NC-2] ADD A TIMELOCK TO CRITICAL FUNCTIONS

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardEscrow.sol

207:   function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {

```

```solidity
File: src/vault/PermissionRegistry.sol

38:   function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

```

```solidity
File: src/vault/Vault.sol

553:     function setFeeRecipient(address _feeRecipient) external onlyOwner {

629:     function setQuitPeriod(uint256 _quitPeriod) external onlyOwner {

```

```solidity
File: src/vault/VaultController.sol

408:   function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

543:   function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {

751:   function setPerformanceFee(uint256 newFee) external onlyOwner {

764:   function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {

791:   function setHarvestCooldown(uint256 newCooldown) external onlyOwner {

804:   function setAdapterHarvestCooldowns(address[] calldata adapters) external onlyOwner {

832:   function setDeploymentController(IDeploymentController _deploymentController) external onlyOwner {

836:   function _setDeploymentController(IDeploymentController _deploymentController) internal {

864:   function setActiveTemplateId(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

```

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

500:     function setHarvestCooldown(uint256 newCooldown) external onlyOwner {

549:     function setPerformanceFee(uint256 newFee) public onlyOwner {

```

### [NC-3] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)

Rather than using `abi.encodePacked` for appending bytes, since version 0.8.4, `bytes.concat()` is enabled.

Since version 0.8.4 for appending bytes, `bytes.concat()` can be used instead of `abi.encodePacked(,)`.

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardEscrow.sol

104:     bytes32 id = keccak256(abi.encodePacked(token, account, amount, nonce));

```

```solidity
File: src/utils/MultiRewardStaking.sol

49:     _name = string(abi.encodePacked("Staked ", IERC20Metadata(address(_stakingToken)).name()));

50:     _symbol = string(abi.encodePacked("pst-", IERC20Metadata(address(_stakingToken)).symbol()));

```

```solidity
File: src/vault/Vault.sol

94:             abi.encodePacked("Popcorn", name(), block.timestamp, "Vault")

```

### [NC-4] SIGNATURE MALLEABILITY OF EVM’S ECRECOVER()

#### Description:

The function calls the Solidity `ecrecover()` function directly to verify the given signatures. However, the `ecrecover()` EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

Although a replay attack seems not possible for this contract, I recommend using the battle-tested OpenZeppelin’s ECDSA library.

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardStaking.sol

459:       address recoveredAddress = ecrecover(

```

```solidity
File: src/vault/Vault.sol

678:             address recoveredAddress = ecrecover(

```

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

646:             address recoveredAddress = ecrecover(

```

#### Recommended Mitigation Steps:

Use the ecrecover function from OpenZeppelin’s ECDSA library for signature verification. (Ensure using a version > 4.7.3 for there was a critical bug >= 4.1.0 < 4.7.3).

### [NC-5] LACK OF EVENT EMISSION AFTER CRITICAL `INITIALIZE()` FUNCTIONS

#### Description:

To record the init parameters for off-chain monitoring and transparency reasons, please consider emitting an event after the initialize() functions:

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardStaking.sol

41:   function initialize(

```

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

46:     function initialize(

```

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

34:     function initialize(

```

### [NC-6] ALLOWS MALLEABLE SECP256K1 SIGNATURES

#### Description:

Here, the `ecrecover()` method doesn’t check the s range.

Homestead ([EIP-2](https://eips.ethereum.org/EIPS/eip-2)) added this limitation, however the precompile remained unaltered. The majority of libraries, including OpenZeppelin, do this check.

Since an order can only be confirmed once and its hash is saved, there doesn’t seem to be a serious danger in existing use cases.

Reference: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/7201e6707f6631d9499a569f492870ebdd4133cf/contracts/utils/cryptography/ECDSA.sol#L138-L149

#### **Proof Of Concept**

[MultiRewardStaking.sol:459-479](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L459-L479)
[Vault.sol:678-700](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L678-L700)
[AdapterBase.sol:646-668](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L646-L668)

### [NC-7] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS

#### Context:

All Contracts

#### Description:

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.

Reference: https://docs.soliditylang.org/en/v0.8.15/natspec-format.html.

#### Recommendation:

NatSpec comments should be increased in contracts

### [NC-8] INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS

#### Context:

All Contracts

#### Description:

If Return parameters are declared, you must prefix them with ”/// @return”.

Some code analysis programs do analysis by reading NatSpec details, if they can’t see the “@return” tag, they do incomplete analysis.

### [NC-9] NO SAME VALUE INPUT CONTROL

#### **Proof Of Concept**

```solidity
File: src/vault/Vault.sol

558:         feeRecipient = _feeRecipient;

633:         quitPeriod = _quitPeriod;

```

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

80:         strategyConfig = _strategyConfig;

81:         harvestCooldown = _harvestCooldown;

```

#### Recommended Mitigation Steps:

Add code like this; if (oracle == \_oracle revert ADDRESS_SAME();

### [NC-10] Omissions in events

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters.

The events should include the new value and old value where possible.

#### **Proof Of Concept**

Events with no old value:

```solidity
File: src/vault/CloneFactory.sol

47:     emit Deployment(clone);

```

```solidity
File: src/vault/CloneRegistry.sol

50:     emit CloneAdded(clone);

```

```solidity
File: src/vault/TemplateRegistry.sol

58:     emit TemplateCategoryAdded(templateCategory);

```

```solidity
File: src/vault/Vault.sol

635:         emit QuitPeriodSet(quitPeriod);

```

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

449:         emit Harvested();

```

### [NC-11] USE A MORE RECENT VERSION OF SOLIDITY

#### **Context:**

All Contracts

#### **Description:**

For security, it is best practice to use the latest Solidity version.

For the security fix list in the versions.

https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Proof Of Concept**

```solidity
File: src/interfaces/IEIP165.sol

2: pragma solidity ^0.8.15;

```

```solidity
File: src/interfaces/IMultiRewardEscrow.sol

3: pragma solidity ^0.8.15;

```

```solidity
File: src/interfaces/IMultiRewardStaking.sol

3: pragma solidity ^0.8.15;

```

```solidity
File: src/interfaces/vault/IAdapter.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/interfaces/vault/IAdminProxy.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/interfaces/vault/ICloneFactory.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/interfaces/vault/ICloneRegistry.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/interfaces/vault/IDeploymentController.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/interfaces/vault/IERC4626.sol

3: pragma solidity ^0.8.15;

```

```solidity
File: src/interfaces/vault/IPermissionRegistry.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/interfaces/vault/IStrategy.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/interfaces/vault/ITemplateRegistry.sol

3: pragma solidity ^0.8.15;

```

```solidity
File: src/interfaces/vault/IVault.sol

3: pragma solidity ^0.8.15;

```

```solidity
File: src/interfaces/vault/IVaultController.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/interfaces/vault/IVaultRegistry.sol

3: pragma solidity ^0.8.15;

```

```solidity
File: src/interfaces/vault/IWithRewards.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/utils/EIP165.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/utils/MultiRewardEscrow.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/utils/MultiRewardStaking.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/vault/AdminProxy.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/vault/CloneFactory.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/vault/CloneRegistry.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/vault/DeploymentController.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/vault/PermissionRegistry.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/vault/TemplateRegistry.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/vault/Vault.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/vault/VaultController.sol

3: pragma solidity ^0.8.15;

```

```solidity
File: src/vault/VaultRegistry.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/vault/adapter/abstracts/OnlyStrategy.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/vault/adapter/abstracts/WithRewards.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/vault/adapter/beefy/IBeefy.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/vault/adapter/yearn/IYearn.sol

4: pragma solidity ^0.8.15;

```

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

4: pragma solidity ^0.8.15;

```

#### **Recommendation**

Old version of Solidity is used, newer version can be used `(0.8.17)`

### [NC-12] For functions, follow solidity standard naming conventions

#### Description:

Solidity’s standard naming convention for internal and private functions: the mixedCase format starting with an underscore (\_mixedCase starting with an underscore)

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardStaking.sol

112:   ) internal override accrueRewards(caller, receiver) {

127:   ) internal override accrueRewards(caller, receiver) {

141:   ) internal override accrueRewards(from, to) {

491:   function computeDomainSeparator() internal view virtual returns (bytes32) {

```

```solidity
File: src/vault/Vault.sol

716:     function computeDomainSeparator() internal view virtual returns (bytes32) {

```

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

684:     function computeDomainSeparator() internal view virtual returns (bytes32) {

```

#### **Recommendation**

Use solidity standard naming convention for internal and private functions

### [NC-13] Lines are too long

#### **Description:**

Usually lines in source code are limited to 80 characters.

[Reference](https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length)

#### **Proof Of Concept**

```solidity
File: src/interfaces/IMultiRewardEscrow.sol

5: import { IERC20Upgradeable as IERC20 } from "openzeppelin-contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";

40:   function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external;

```

```solidity
File: src/interfaces/IMultiRewardStaking.sol

48:   function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external;

58:   function rewardInfos(IERC20 rewardToken) external view returns (RewardInfo memory);

60:   function escrowInfos(IERC20 rewardToken) external view returns (EscrowInfo memory);

```

```solidity
File: src/interfaces/vault/IAdapter.sol

20:     function supportsInterface(bytes4 interfaceId) external view returns (bool);

```

```solidity
File: src/interfaces/vault/IAdminProxy.sol

9:   function execute(address target, bytes memory callData) external returns (bool, bytes memory);

```

```solidity
File: src/interfaces/vault/ICloneFactory.sol

10:   function deploy(Template memory template, bytes memory data) external returns (address);

```

```solidity
File: src/interfaces/vault/IDeploymentController.sol

12:   function templateCategoryExists(bytes32 templateCategory) external view returns (bool);

24:   function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external;

26:   function getTemplate(bytes32 templateCategory, bytes32 templateId) external view returns (Template memory);

```

```solidity
File: src/interfaces/vault/IERC4626.sol

5: import { IERC20Upgradeable as IERC20 } from "openzeppelin-contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";

8:   event Deposit(address indexed sender, address indexed owner, uint256 assets, uint256 shares);

26:   function convertToShares(uint256 assets) external view returns (uint256 shares);

28:   function convertToAssets(uint256 shares) external view returns (uint256 assets);

32:   function maxDeposit(address receiver) external view returns (uint256 maxAssets);

34:   function previewDeposit(uint256 assets) external view returns (uint256 shares);

36:   function deposit(uint256 assets, address receiver) external returns (uint256 shares);

44:   function mint(uint256 shares, address receiver) external returns (uint256 assets);

48:   function maxWithdraw(address owner) external view returns (uint256 maxAssets);

50:   function previewWithdraw(uint256 assets) external view returns (uint256 shares);

52:   function withdraw(uint256 assets, address receiver, address owner) external returns (uint256 shares);

58:   function previewRedeem(uint256 shares) external view returns (uint256 assets);

60:   function redeem(uint256 shares, address receiver, address owner) external returns (uint256 assets);

```

```solidity
File: src/interfaces/vault/IPermissionRegistry.sol

14:   function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external;

```

```solidity
File: src/interfaces/vault/ITemplateRegistry.sol

24:   function templates(bytes32 templateCategory, bytes32 templateId) external view returns (Template memory);

26:   function templateCategoryExists(bytes32 templateCategory) external view returns (bool);

32:   function getTemplate(bytes32 templateCategory, bytes32 templateId) external view returns (Template memory);

34:   function getTemplateIds(bytes32 templateCategory) external view returns (bytes32[] memory);

44:   function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external;

```

```solidity
File: src/interfaces/vault/IVaultController.sol

39:   function proposeVaultAdapters(address[] memory vaults, IERC4626[] memory newAdapter) external;

43:   function proposeVaultFees(address[] memory vaults, VaultFees[] memory newFees) external;

47:   function registerVaults(address[] memory vaults, VaultMetadata[] memory metadata) external;

55:   function addStakingRewardsTokens(address[] memory vaults, bytes[] memory rewardsTokenData) external;

69:   function setEscrowTokenFees(IERC20[] memory tokens, uint256[] memory fees) external;

73:   function toggleTemplateEndorsements(bytes32[] memory templateCategories, bytes32[] memory templateIds) external;

99:   function setDeploymentController(IDeploymentController _deploymentController) external;

101:   function setActiveTemplateId(bytes32 templateCategory, bytes32 templateId) external;

103:   function activeTemplateId(bytes32 templateCategory) external view returns (bytes32);

```

```solidity
File: src/interfaces/vault/IVaultRegistry.sol

25:   function getVault(address vault) external view returns (VaultMetadata memory);

```

```solidity
File: src/utils/EIP165.sol

7:   function supportsInterface(bytes4 interfaceId) public view virtual returns (bool) {}

```

```solidity
File: src/utils/MultiRewardEscrow.sol

6: import { SafeERC20Upgradeable as SafeERC20 } from "openzeppelin-contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

7: import { IERC20Upgradeable as IERC20 } from "openzeppelin-contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";

38:   function getEscrowIdsByUser(address account) external view returns (bytes32[] memory) {

42:   function getEscrowIdsByUserAndToken(address account, IERC20 token) external view returns (bytes32[] memory) {

51:   function getEscrows(bytes32[] calldata escrowIds) external view returns (Escrow[] memory) {

73:   event Locked(IERC20 indexed token, address indexed account, uint256 amount, uint32 duration, uint32 offset);

136:   event RewardsClaimed(IERC20 indexed token, address indexed account, uint256 amount);

141:     return escrows[escrowId].lastUpdateTime != 0 && escrows[escrowId].balance > 0;

144:   function getClaimableAmount(bytes32 escrowId) external view returns (uint256) {

170:   function _getClaimableAmount(Escrow memory escrow) internal view returns (uint256) {

207:   function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {

208:     if (tokens.length != tokenFees.length) revert ArraysNotMatching(tokens.length, tokenFees.length);

```

```solidity
File: src/utils/MultiRewardStaking.sol

6: import { SafeERC20Upgradeable as SafeERC20 } from "openzeppelin-contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

7: import { ERC4626Upgradeable, ERC20Upgradeable, IERC20Upgradeable as IERC20, IERC20MetadataUpgradeable as IERC20Metadata } from "openzeppelin-contracts-upgradeable/token/ERC20/extensions/ERC4626Upgradeable.sol";

8: import { MathUpgradeable as Math } from "openzeppelin-contracts-upgradeable/utils/math/MathUpgradeable.sol";

49:     _name = string(abi.encodePacked("Staked ", IERC20Metadata(address(_stakingToken)).name()));

50:     _symbol = string(abi.encodePacked("pst-", IERC20Metadata(address(_stakingToken)).symbol()));

59:   function name() public view override(ERC20Upgradeable, IERC20Metadata) returns (string memory) {

63:   function symbol() public view override(ERC20Upgradeable, IERC20Metadata) returns (string memory) {

67:   function decimals() public view override(ERC20Upgradeable, IERC20Metadata) returns (uint8) {

98:   function _convertToShares(uint256 assets, Math.Rounding) internal pure override returns (uint256) {

102:   function _convertToAssets(uint256 shares, Math.Rounding) internal pure override returns (uint256) {

128:     if (caller != owner) _approve(owner, msg.sender, allowance(owner, msg.sender) - shares);

142:     if (from == address(0) || to == address(0)) revert ZeroAddressTransfer(from, to);

159:   event RewardsClaimed(address indexed user, IERC20 rewardToken, uint256 amount, bool escrowed);

170:   function claimRewards(address user, IERC20[] memory _rewardTokens) external accrueRewards(msg.sender, user) {

197:     uint256 escrowed = rewardAmount.mulDiv(uint256(escrowInfo.escrowPercentage), 1e18, Math.Rounding.Down);

201:     escrow.lock(rewardToken, user, escrowed, escrowInfo.escrowDuration, escrowInfo.offset);

220:   event RewardInfoUpdate(IERC20 rewardToken, uint160 rewardsPerSecond, uint32 rewardsEndTimestamp);

252:     if (asset() == address(rewardToken)) revert RewardTokenCantBeStakingToken();

255:     if (rewards.lastUpdatedTimestamp > 0) revert RewardTokenAlreadyExist(rewardToken);

274:     uint64 ONE = (10**IERC20Metadata(address(rewardToken)).decimals()).safeCastTo64();

296:   function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {

300:     if (rewards.lastUpdatedTimestamp == 0) revert RewardTokenDoesntExist(rewardToken);

309:       prevEndTime > block.timestamp ? prevEndTime : block.timestamp.safeCastTo32(),

331:     if (rewards.lastUpdatedTimestamp == 0) revert RewardTokenDoesntExist(rewardToken);

336:     uint256 accrued = rewards.rewardsPerSecond == 0 ? amount : _accrueStatic(rewards);

342:       rewardsEndTimestamp = _calcRewardsEnd(rewards.rewardsEndTimestamp, rewards.rewardsPerSecond, amount);

346:     rewardInfos[rewardToken].lastUpdatedTimestamp = block.timestamp.safeCastTo32();

348:     emit RewardInfoUpdate(rewardToken, rewards.rewardsPerSecond, rewardsEndTimestamp);

357:       amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);

359:     return (block.timestamp + (amount / uint256(rewardsPerSecond))).safeCastTo32();

377:       if (rewards.rewardsPerSecond > 0) _accrueRewards(rewardToken, _accrueStatic(rewards));

390:   function _accrueStatic(RewardInfo memory rewards) internal view returns (uint256 accrued) {

406:       deltaIndex = accrued.mulDiv(uint256(10**decimals()), supplyTokens, Math.Rounding.Down).safeCastTo224();

409:     rewardInfos[_rewardToken].lastUpdatedTimestamp = block.timestamp.safeCastTo32();

427:     uint256 supplierDelta = balanceOf(_user).mulDiv(deltaIndex, uint256(rewards.ONE), Math.Rounding.Down);

466:                 keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),

481:       if (recoveredAddress == address(0) || recoveredAddress != owner) revert InvalidSigner(recoveredAddress);

488:     return block.chainid == INITIAL_CHAIN_ID ? INITIAL_DOMAIN_SEPARATOR : computeDomainSeparator();

495:           keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),

```

```solidity
File: src/vault/CloneFactory.sol

39:   function deploy(Template calldata template, bytes calldata data) external onlyOwner returns (address clone) {

```

```solidity
File: src/vault/CloneRegistry.sol

57:   function getClonesByCategoryAndId(bytes32 templateCategory, bytes32 templateId)

```

```solidity
File: src/vault/DeploymentController.sol

10: import { ITemplateRegistry, Template } from "../interfaces/vault/ITemplateRegistry.sol";

80:   function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

104:     Template memory template = templateRegistry.getTemplate(templateCategory, templateId);

```

```solidity
File: src/vault/PermissionRegistry.sol

38:   function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

43:       if (newPermissions[i].endorsed && newPermissions[i].rejected) revert Mismatch();

45:       emit PermissionSet(targets[i], newPermissions[i].endorsed, newPermissions[i].rejected);

```

```solidity
File: src/vault/TemplateRegistry.sol

39:   event TemplateAdded(bytes32 templateCategory, bytes32 templateId, address implementation);

53:     if (templateCategoryExists[templateCategory]) revert TemplateCategoryExists(templateCategory);

72:     if (!templateCategoryExists[templateCategory]) revert KeyNotFound(templateCategory);

102:   function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

108:     emit TemplateEndorsementToggled(templateCategory, templateId, oldEndorsement, !oldEndorsement);

119:   function getTemplateIds(bytes32 templateCategory) external view returns (bytes32[] memory) {

123:   function getTemplate(bytes32 templateCategory, bytes32 templateId) external view returns (Template memory) {

```

```solidity
File: src/vault/Vault.sol

6: import {SafeERC20Upgradeable as SafeERC20} from "openzeppelin-contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

7: import {ReentrancyGuardUpgradeable} from "openzeppelin-contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";

8: import {PausableUpgradeable} from "openzeppelin-contracts-upgradeable/security/PausableUpgradeable.sol";

10: import {IERC20Metadata} from "openzeppelin-contracts/token/ERC20/extensions/IERC20Metadata.sol";

12: import {MathUpgradeable as Math} from "openzeppelin-contracts-upgradeable/utils/math/MathUpgradeable.sol";

14: import {ERC20Upgradeable} from "openzeppelin-contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";

295:         uint256 supply = totalSupply(); // Saves an extra SLOAD if totalSupply is non-zero.

309:         uint256 supply = totalSupply(); // Saves an extra SLOAD if totalSupply is non-zero.

514:     event FeeRecipientUpdated(address oldFeeRecipient, address newFeeRecipient);

686:                                     "Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"

721:                         "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"

```

```solidity
File: src/vault/VaultController.sol

5: import { SafeERC20Upgradeable as SafeERC20 } from "openzeppelin-contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

7: import { IVault, VaultInitParams, VaultFees } from "../interfaces/vault/IVault.sol";

10: import { IDeploymentController, ICloneRegistry } from "../interfaces/vault/IDeploymentController.sol";

11: import { ITemplateRegistry, Template } from "../interfaces/vault/ITemplateRegistry.sol";

12: import { IPermissionRegistry, Permission } from "../interfaces/vault/IPermissionRegistry.sol";

13: import { IVaultRegistry, VaultMetadata } from "../interfaces/vault/IVaultRegistry.sol";

40:   bytes4 internal immutable DEPLOY_SIG = bytes4(keccak256("deploy(bytes32,bytes32,bytes)"));

76:   event VaultDeployed(address indexed vault, address indexed staking, address indexed adapter);

104:       vaultData.adapter = IERC4626(_deployAdapter(vaultData.asset, adapterData, strategyData, _deploymentController));

108:     if (staking == address(0)) staking = _deployStaking(IERC20(address(vault)), _deploymentController);

116:     _handleInitialDeposit(initialDeposit, IERC20(vaultData.asset), IERC4626(vault));

120:   function _deployVault(VaultInitParams memory vaultData, IDeploymentController _deploymentController)

154:   function _handleVaultStakingRewards(address vault, bytes memory rewardsData) internal {

194:     adapter = _deployAdapter(asset, adapterData, strategyData, deploymentController);

213:       requiredSigs = templateRegistry.getTemplate(STRATEGY, strategyData.id).requiredSigs;

219:         abi.encode(asset, address(adminProxy), IStrategy(strategy), harvestCooldown, requiredSigs, strategyData.data),

232:       abi.encodeWithSelector(DEPLOY_SIG, ADAPTER, adapterData.id, _encodeAdapterData(adapterData, baseAdapterData))

238:     adminProxy.execute(adapter, abi.encodeWithSelector(IAdapter.setPerformanceFee.selector, performanceFee));

242:   function _encodeAdapterData(DeploymentArgs memory adapterData, bytes memory baseAdapterData)

256:   function _deployStrategy(DeploymentArgs memory strategyData, IDeploymentController _deploymentController)

284:   function _deployStaking(IERC20 asset, IDeploymentController _deploymentController)

294:         abi.encodeWithSelector(IMultiRewardStaking.initialize.selector, asset, escrow, adminProxy)

313:   function proposeVaultAdapters(address[] calldata vaults, IERC4626[] calldata newAdapter) external {

321:       if (!_cloneRegistry.cloneExists(address(newAdapter[i]))) revert DoesntExist(address(newAdapter[i]));

352:   function proposeVaultFees(address[] calldata vaults, VaultFees[] calldata fees) external {

390:   function _registerVault(address vault, VaultMetadata memory metadata) internal {

408:   function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

412:       abi.encodeWithSelector(IPermissionRegistry.setPermissions.selector, targets, newPermissions)

433:   function addStakingRewardsTokens(address[] memory vaults, bytes[] memory rewardTokenData) public {

446:       ) = abi.decode(rewardTokenData[i], (address, uint160, uint256, bool, uint224, uint24, uint256));

452:         abi.encodeWithSelector(IERC20.approve.selector, staking, type(uint256).max)

457:       IERC20(rewardsToken).transferFrom(msg.sender, address(adminProxy), amount);

499:         abi.encodeWithSelector(IMultiRewardStaking.changeRewardSpeed.selector, rewardTokens[i], rewardsSpeeds[i])

543:   function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {

561:   function addTemplateCategories(bytes32[] calldata templateCategories) external onlyOwner {

567:         abi.encodeWithSelector(IDeploymentController.addTemplateCategory.selector, templateCategories[i])

579:   function toggleTemplateEndorsements(bytes32[] calldata templateCategories, bytes32[] calldata templateIds)

667:   function _verifyCreatorOrOwner(address vault) internal returns (VaultMetadata memory metadata) {

669:     if (msg.sender != metadata.creator || msg.sender != owner) revert NotSubmitterNorOwner(msg.sender);

673:   function _verifyCreator(address vault) internal view returns (VaultMetadata memory metadata) {

692:   function _verifyAdapterConfiguration(address adapter, bytes32 adapterId) internal view {

700:   function _verifyEqualArrayLength(uint256 length1, uint256 length2) internal pure {

764:   function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {

769:         abi.encodeWithSelector(IAdapter.setPerformanceFee.selector, performanceFee)

804:   function setAdapterHarvestCooldowns(address[] calldata adapters) external onlyOwner {

809:         abi.encodeWithSelector(IAdapter.setHarvestCooldown.selector, harvestCooldown)

824:   event DeploymentControllerChanged(address oldController, address newController);

832:   function setDeploymentController(IDeploymentController _deploymentController) external onlyOwner {

836:   function _setDeploymentController(IDeploymentController _deploymentController) internal {

837:     if (address(_deploymentController) == address(0) || address(deploymentController) == address(_deploymentController))

840:     emit DeploymentControllerChanged(address(deploymentController), address(_deploymentController));

864:   function setActiveTemplateId(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

```

```solidity
File: src/vault/VaultRegistry.sol

45:     if (metadata[_metadata.vault].vault != address(0)) revert VaultAlreadyRegistered();

59:   function getVault(address vault) external view returns (VaultMetadata memory) {

63:   function getVaultsByAsset(address asset) external view returns (address[] memory) {

75:   function getSubmitter(address vault) external view returns (VaultMetadata memory) {

```

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

6: import {ERC4626Upgradeable, IERC20Upgradeable as IERC20, IERC20MetadataUpgradeable as IERC20Metadata, ERC20Upgradeable as ERC20} from "openzeppelin-contracts-upgradeable/token/ERC20/extensions/ERC4626Upgradeable.sol";

7: import {SafeERC20Upgradeable as SafeERC20} from "openzeppelin-contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

8: import {ReentrancyGuardUpgradeable} from "openzeppelin-contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";

9: import {MathUpgradeable as Math} from "openzeppelin-contracts-upgradeable/utils/math/MathUpgradeable.sol";

10: import {PausableUpgradeable} from "openzeppelin-contracts-upgradeable/security/PausableUpgradeable.sol";

594:     function _protocolDeposit(uint256 assets, uint256 shares) internal virtual {

654:                                     "Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"

689:                         "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"

```

```solidity
File: src/vault/adapter/abstracts/WithRewards.sol

21:   function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {

22:     return interfaceId == type(IWithRewards).interfaceId || interfaceId == type(IAdapter).interfaceId;

```

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

6: import {AdapterBase, IERC20, IERC20Metadata, SafeERC20, ERC20, Math, IStrategy, IAdapter} from "../abstracts/AdapterBase.sol";

8: import {IBeefyVault, IBeefyBooster, IBeefyBalanceCheck, IBeefyStrat} from "./IBeefy.sol";

9: import {IPermissionRegistry} from "../../../interfaces/vault/IPermissionRegistry.sol";

```

```solidity
File: src/vault/adapter/yearn/IYearn.sol

6: import { IERC20Upgradeable as IERC20 } from "openzeppelin-contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";

```

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

6: import {AdapterBase, ERC4626Upgradeable as ERC4626, IERC20, IERC20Metadata, ERC20, SafeERC20, Math, IStrategy, IAdapter} from "../abstracts/AdapterBase.sol";

45:         yVault = VaultAPI(IYearnRegistry(externalRegistry).latestVault(_asset));

```

#### **Recommendation**

The lines should be split when they reach that length.

### [NC-14] NOT VERIFIED INPUT

External / public functions parameters should be validated to make sure the address is not 0.

Otherwise if not given the right input it can mistakenly lead to loss of user funds.

#### **Proof Of Concept**

```solidity
File: src/vault/DeploymentController.sol

121:   function nominateNewDependencyOwner(address _owner) external onlyOwner {

```

```solidity
File: src/vault/VaultController.sol

723:   function nominateNewAdminProxyOwner(address newOwner) external onlyOwner {

```

## Low Issues

|      | Issue                                                                      |
| ---- | :------------------------------------------------------------------------- |
| L-1  | DoS with block gas limit                                                   |
| L-2  | Avoid transfer()/send() as reentrancy mitigations                          |
| L-3  | CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE                             |
| L-4  | The critical parameters in initialize(...) are not set safely              |
| L-5  | DoS with Failed Call                                                       |
| L-6  | `INITIALIZE()` FUNCTION CAN BE CALLED BY ANYBODY                           |
| L-7  | INSUFFICIENT COVERAGE                                                      |
| L-8  | LACK OF INPUT VALIDATION                                                   |
| L-9  | LOSS OF PRECISION DUE TO ROUNDING                                          |
| L-10 | IMPLMENTATION IS NOT FULLY UP TO EIP-4626’S SPECIFICATION                  |
| L-11 | MISSING CALLS TO \_\_REENTRANCYGUARD_INIT FUNCTIONS OF INHERITED CONTRACTS |
| L-12 | SOLMATE’S SAFETRANSFERLIB DOESN’T CHECK WHETHER THE ERC20 CONTRACT EXISTS  |
| L-13 | Modifier side-effects                                                      |
| L-14 | A SINGLE POINT OF FAILURE                                                  |
| L-15 | Block values as a proxy for time                                           |
| L-16 | Account existence check for low-level calls                                |
| L-17 | UNSAFE CAST                                                                |
| L-18 | NOT USING THE LATEST VERSION OF OPENZEPPELIN FROM DEPENDENCIES             |
| L-19 | Unspecific compiler version pragma                                         |

### [L-1] DoS with block gas limit

Programming patterns such as looping over arrays of unknown size may lead to DoS when the gas cost of execution exceeds the block gas limit.

When smart contracts are deployed or functions inside them are called, the execution of these actions always requires a certain amount of gas, based of how much computation is needed to complete them. The Ethereum network specifies a block gas limit and the sum of all transactions included in a block can not exceed the threshold.

Programming patterns that are harmless in centralized applications can lead to Denial of Service conditions in smart contracts when the cost of executing a function exceeds the block gas limit. Modifying an array of unknown size, that increases in size over time, can lead to such a Denial of Service condition.

Reference: https://swcregistry.io/docs/SWC-128

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardEscrow.sol

51:   function getEscrows(bytes32[] calldata escrowIds) external view returns (Escrow[] memory) {
          for (uint256 i = 0; i < escrowIds.length; i++) {

154:   function claimRewards(bytes32[] memory escrowIds) external {
155:        for (uint256 i = 0; i < escrowIds.length; i++) {

207:   function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {
210:        for (uint256 i = 0; i < tokens.length; i++) {
```

```solidity
File: src/utils/MultiRewardStaking.sol

170:   function claimRewards(address user, IERC20[] memory _rewardTokens) external accrueRewards(msg.sender, user) {
171:        for (uint8 i; i < _rewardTokens.length; i++) {

372:     IERC20[] memory _rewardTokens = rewardTokens;
373:        for (uint8 i; i < _rewardTokens.length; i++) {

```

```solidity
File: src/vault/PermissionRegistry.sol

38:   function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {
39:     uint256 len = targets.length;
42:     for (uint256 i = 0; i < len; i++) {

```

```solidity
File: src/vault/VaultController.sol

313:   function proposeVaultAdapters(address[] calldata vaults, IERC4626[] calldata newAdapter) external {
            uint8 len = uint8(vaults.length);

            _verifyEqualArrayLength(len, newAdapter.length);

            ICloneRegistry _cloneRegistry = cloneRegistry;
            for (uint8 i = 0; i < len; i++) {

335:   function changeVaultAdapters(address[] calldata vaults) external {
            uint8 len = uint8(vaults.length);
            for (uint8 i = 0; i < len; i++) {

352:   function proposeVaultFees(address[] calldata vaults, VaultFees[] calldata fees) external {
            uint8 len = uint8(vaults.length);

            _verifyEqualArrayLength(len, fees.length);

            for (uint8 i = 0; i < len; i++) {

433:    function addStakingRewardsTokens(address[] memory vaults, bytes[] memory rewardTokenData) public {
            _verifyEqualArrayLength(vaults.length, rewardTokenData.length);
            address staking;
            uint8 len = uint8(vaults.length);
            for (uint256 i = 0; i < len; i++) {

483:     function changeStakingRewardsSpeeds(
                address[] calldata vaults,
                IERC20[] calldata rewardTokens,
                uint160[] calldata rewardsSpeeds
            ) external {
                uint8 len = uint8(vaults.length);

                _verifyEqualArrayLength(len, rewardTokens.length);
                _verifyEqualArrayLength(len, rewardsSpeeds.length);

                address staking;
                for (uint256 i = 0; i < len; i++) {

512:     function fundStakingRewards(
                address[] calldata vaults,
                IERC20[] calldata rewardTokens,
                uint256[] calldata amounts
            ) external {
                uint8 len = uint8(vaults.length);

                _verifyEqualArrayLength(len, rewardTokens.length);
                _verifyEqualArrayLength(len, amounts.length);

                address staking;
                for (uint256 i = 0; i < len; i++) {

561:     function addTemplateCategories(bytes32[] calldata templateCategories) external onlyOwner {
                address _deploymentController = address(deploymentController);
                uint8 len = uint8(templateCategories.length);
                for (uint256 i = 0; i < len; i++) {

579:     function toggleTemplateEndorsements(bytes32[] calldata templateCategories, bytes32[] calldata templateIds)
                external
                onlyOwner
            {
                uint8 len = uint8(templateCategories.length);
                _verifyEqualArrayLength(len, templateIds.length);

                address _deploymentController = address(deploymentController);
                for (uint256 i = 0; i < len; i++) {

605:     function pauseAdapters(address[] calldata vaults) external {
                uint8 len = uint8(vaults.length);
                for (uint256 i = 0; i < len; i++) {

618:     function pauseVaults(address[] calldata vaults) external {
                uint8 len = uint8(vaults.length);
                for (uint256 i = 0; i < len; i++) {

631:     function unpauseAdapters(address[] calldata vaults) external {
                uint8 len = uint8(vaults.length);
                for (uint256 i = 0; i < len; i++) {

644:     function unpauseVaults(address[] calldata vaults) external {
                uint8 len = uint8(vaults.length);
                for (uint256 i = 0; i < len; i++) {

764:     function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {
                uint8 len = uint8(adapters.length);
                for (uint256 i = 0; i < len; i++) {

804:     function setAdapterHarvestCooldowns(address[] calldata adapters) external onlyOwner {
                uint8 len = uint8(adapters.length);
                for (uint256 i = 0; i < len; i++) {

```

#### Recommended Mitigation Steps:

Caution is advised when you expect to have large arrays that grow over time. Actions that require looping across the entire data structure should be avoided.

If you absolutely must loop over an array of unknown size, then you should plan for it to potentially take multiple blocks, and therefore require multiple transactions.

### [L-2] Avoid `transfer()`/`send()` as reentrancy mitigations

#### Description:

Although `transfer()` and `send()` have been recommended as a security best-practice to prevent reentrancy attacks because they only forward 2300 gas, the gas repricing of opcodes may break deployed contracts.

The `transfer()` and `send()` functions forward a fixed amount of 2300 gas. Historically, it has often been recommended to use these functions for value transfers to guard against reentrancy attacks. However, the gas cost of EVM instructions may change significantly during hard forks which may break already deployed contract systems that make fixed assumptions about gas costs.

Reference: https://swcregistry.io/docs/SWC-134

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardStaking.sol

182:         _rewardTokens[i].transfer(user, rewardAmount);

```

#### Recommended Mitigation Steps:

Avoid the use of `transfer()` and `send()` and do not otherwise specify a fixed amount of gas when performing calls. Use `.call.value(...)("")` instead. Use the checks-effects-interactions pattern and/or reentrancy locks to prevent reentrancy attacks.

### [L-3] CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE

The critical procedures should be two step process.

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardEscrow.sol

207:   function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {

```

```solidity
File: src/vault/PermissionRegistry.sol

38:   function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

```

```solidity
File: src/vault/Vault.sol

553:     function setFeeRecipient(address _feeRecipient) external onlyOwner {

629:     function setQuitPeriod(uint256 _quitPeriod) external onlyOwner {

```

```solidity
File: src/vault/VaultController.sol

408:   function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

543:   function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {

751:   function setPerformanceFee(uint256 newFee) external onlyOwner {

764:   function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {

791:   function setHarvestCooldown(uint256 newCooldown) external onlyOwner {

804:   function setAdapterHarvestCooldowns(address[] calldata adapters) external onlyOwner {

832:   function setDeploymentController(IDeploymentController _deploymentController) external onlyOwner {

836:   function _setDeploymentController(IDeploymentController _deploymentController) internal {

864:   function setActiveTemplateId(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

```

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

500:     function setHarvestCooldown(uint256 newCooldown) external onlyOwner {

549:     function setPerformanceFee(uint256 newFee) public onlyOwner {

```

#### Recommended Mitigation Steps:

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

### [L-4] The critical parameters in `initialize(...)` are not set safely

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardStaking.sol

53:   escrow = _escrow;

```

```solidity
File: src/vault/Vault.sol

77:     asset = asset_;
78:     adapter = adapter_;
88:     fees = fees_;

```

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

46:     function initialize(

```

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

34:     function initialize(

```

### [L-5] DoS with Failed Call

External calls can fail accidentally or deliberately, which can cause a DoS condition in the contract. To minimize the damage caused by such failures, it is better to isolate each external call into its own transaction that can be initiated by the recipient of the call. This is especially relevant for payments, where it is better to let users withdraw funds rather than push funds to them automatically (this also reduces the chance of problems with the gas limit).

Reference: https://swcregistry.io/docs/SWC-113

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardStaking.sol

182:         _rewardTokens[i].transfer(user, rewardAmount);

```

#### Recommended Mitigation Steps:

It is recommended to follow call best practices:

- Avoid combining multiple calls in a single transaction, especially when calls are executed as part of a loop
- Always assume that external calls can fail
- Implement the contract logic to handle failed calls

### [L-6] `INITIALIZE()` FUNCTION CAN BE CALLED BY ANYBODY

#### Description:

`initialize()` function can be called anybody when the contract is not initialized.

More importantly, if someone else runs this function, they will have full authority because of the `__Ownable_init()` function. Also, there is no 0 address check in the address arguments of the initialize() function, which must be defined.

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardStaking.sol

41:   function initialize(
            IERC20 _stakingToken,
            IMultiRewardEscrow _escrow,
            address _owner
        ) external initializer {
            __ERC4626_init(IERC20Metadata(address(_stakingToken)));
            __Owned_init(_owner);

            _name = string(abi.encodePacked("Staked ", IERC20Metadata(address(_stakingToken)).name()));
            _symbol = string(abi.encodePacked("pst-", IERC20Metadata(address(_stakingToken)).symbol()));
            _decimals = IERC20Metadata(address(_stakingToken)).decimals();

            escrow = _escrow;

            INITIAL_CHAIN_ID = block.chainid;
            INITIAL_DOMAIN_SEPARATOR = computeDomainSeparator();
        }

```

```solidity
File: src/vault/Vault.sol

57:         function initialize(
                IERC20 asset_,
                IERC4626 adapter_,
                VaultFees calldata fees_,
                address feeRecipient_,
                address owner
            ) external initializer {
                __ERC20_init(
                    string.concat(
                        "Popcorn ",
                        IERC20Metadata(address(asset_)).name(),
                        " Vault"
                    ),
                    string.concat("pop-", IERC20Metadata(address(asset_)).symbol())
                );
                __Owned_init(owner);

                if (address(asset_) == address(0)) revert InvalidAsset();
                if (address(asset_) != adapter_.asset()) revert InvalidAdapter();

                asset = asset_;
                adapter = adapter_;

                asset.approve(address(adapter_), type(uint256).max);

                _decimals = IERC20Metadata(address(asset_)).decimals();

                INITIAL_CHAIN_ID = block.chainid;
                INITIAL_DOMAIN_SEPARATOR = computeDomainSeparator();

                feesUpdatedAt = block.timestamp;
                fees = fees_;

                if (feeRecipient_ == address(0)) revert InvalidFeeRecipient();
                feeRecipient = feeRecipient_;

                contractName = keccak256(
                    abi.encodePacked("Popcorn", name(), block.timestamp, "Vault")
                );

                emit VaultInitialized(contractName, address(asset));
            }

```

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

46:         function initialize(
                bytes memory adapterInitData,
                address registry,
                bytes memory beefyInitData
            ) external initializer {
                (address _beefyVault, address _beefyBooster) = abi.decode(
                    beefyInitData,
                    (address, address)
                );
                __AdapterBase_init(adapterInitData);

                if (!IPermissionRegistry(registry).endorsed(_beefyVault))
                    revert NotEndorsed(_beefyVault);
                if (IBeefyVault(_beefyVault).want() != asset())
                    revert InvalidBeefyVault(_beefyVault);
                if (
                    _beefyBooster != address(0) &&
                    IBeefyBooster(_beefyBooster).stakedToken() != _beefyVault
                ) revert InvalidBeefyBooster(_beefyBooster);

                _name = string.concat(
                    "Popcorn Beefy",
                    IERC20Metadata(asset()).name(),
                    " Adapter"
                );
                _symbol = string.concat("popB-", IERC20Metadata(asset()).symbol());

                beefyVault = IBeefyVault(_beefyVault);
                beefyBooster = IBeefyBooster(_beefyBooster);

                beefyBalanceCheck = IBeefyBalanceCheck(
                    _beefyBooster == address(0) ? _beefyVault : _beefyBooster
                );

                IERC20(asset()).approve(_beefyVault, type(uint256).max);

                if (_beefyBooster != address(0))
                    IERC20(_beefyVault).approve(_beefyBooster, type(uint256).max);
            }

```

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

34:         function initialize(
                bytes memory adapterInitData,
                address externalRegistry,
                bytes memory
            ) external initializer {
                (address _asset, , , , , ) = abi.decode(
                    adapterInitData,
                    (address, address, address, uint256, bytes4[8], bytes)
                );
                __AdapterBase_init(adapterInitData);

                yVault = VaultAPI(IYearnRegistry(externalRegistry).latestVault(_asset));

                _name = string.concat(
                    "Popcorn Yearn",
                    IERC20Metadata(asset()).name(),
                    " Adapter"
                );
                _symbol = string.concat("popY-", IERC20Metadata(asset()).symbol());

                IERC20(_asset).approve(address(yVault), type(uint256).max);
            }

```

### [L-7] INSUFFICIENT COVERAGE

Testing all functions is best practice in terms of security criteria.

#### **Proof Of Concept**

| File                                                                                | % Lines           | % Statements      | % Branches       | % Funcs          |
| ----------------------------------------------------------------------------------- | ----------------- | ----------------- | ---------------- | ---------------- |
| node_modules/@openzeppelin/contracts/proxy/Clones.sol                               | 0.00% (0/6)       | 0.00% (0/6)       | 0.00% (0/4)      | 0.00% (0/4)      |
| node_modules/@openzeppelin/contracts/token/ERC20/ERC20.sol                          | 83.64% (46/55)    | 80.65% (50/62)    | 45.45% (10/22)   | 77.78% (14/18)   |
| node_modules/@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol                | 0.00% (0/17)      | 0.00% (0/23)      | 0.00% (0/10)     | 0.00% (0/7)      |
| node_modules/@openzeppelin/contracts/utils/Address.sol                              | 0.00% (0/26)      | 0.00% (0/28)      | 0.00% (0/16)     | 0.00% (0/13)     |
| node_modules/@openzeppelin/contracts/utils/Context.sol                              | 50.00% (1/2)      | 50.00% (1/2)      | 100.00% (0/0)    | 50.00% (1/2)     |
| node_modules/@openzeppelin/contracts/utils/Strings.sol                              | 0.00% (0/18)      | 0.00% (0/25)      | 0.00% (0/4)      | 0.00% (0/4)      |
| node_modules/@openzeppelin/contracts/utils/math/Math.sol                            | 6.09% (7/115)     | 5.69% (7/123)     | 2.08% (1/48)     | 14.29% (2/14)    |
| node_modules/openzeppelin-upgradeable/proxy/utils/Initializable.sol                 | 0.00% (0/4)       | 0.00% (0/4)       | 0.00% (0/4)      | 0.00% (0/1)      |
| node_modules/openzeppelin-upgradeable/security/PausableUpgradeable.sol              | 100.00% (9/9)     | 100.00% (9/9)     | 75.00% (3/4)     | 100.00% (7/7)    |
| node_modules/openzeppelin-upgradeable/security/ReentrancyGuardUpgradeable.sol       | 0.00% (0/2)       | 0.00% (0/2)       | 100.00% (0/0)    | 0.00% (0/2)      |
| node_modules/openzeppelin-upgradeable/token/ERC20/ERC20Upgradeable.sol              | 68.97% (40/58)    | 67.69% (44/65)    | 31.82% (7/22)    | 80.00% (16/20)   |
| node_modules/openzeppelin-upgradeable/token/ERC20/extensions/ERC4626Upgradeable.sol | 74.42% (32/43)    | 75.51% (37/49)    | 40.00% (4/10)    | 56.52% (13/23)   |
| node_modules/openzeppelin-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol    | 0.00% (0/17)      | 0.00% (0/23)      | 0.00% (0/10)     | 0.00% (0/7)      |
| node_modules/openzeppelin-upgradeable/utils/AddressUpgradeable.sol                  | 0.00% (0/19)      | 0.00% (0/21)      | 0.00% (0/14)     | 0.00% (0/9)      |
| node_modules/openzeppelin-upgradeable/utils/ContextUpgradeable.sol                  | 50.00% (1/2)      | 50.00% (1/2)      | 100.00% (0/0)    | 25.00% (1/4)     |
| node_modules/openzeppelin-upgradeable/utils/math/MathUpgradeable.sol                | 0.00% (0/69)      | 0.00% (0/73)      | 0.00% (0/24)     | 0.00% (0/8)      |
| src/utils/EIP165.sol                                                                | 100.00% (0/0)     | 100.00% (0/0)     | 100.00% (0/0)    | 0.00% (0/1)      |
| src/utils/MultiRewardEscrow.sol                                                     | 100.00% (42/42)   | 88.33% (53/60)    | 55.56% (10/18)   | 100.00% (9/9)    |
| src/utils/MultiRewardStaking.sol                                                    | 100.00% (110/110) | 89.78% (123/137)  | 63.04% (29/46)   | 100.00% (26/26)  |
| src/utils/Owned.sol                                                                 | 100.00% (7/7)     | 100.00% (7/7)     | 75.00% (3/4)     | 100.00% (3/3)    |
| src/utils/OwnedUpgradeable.sol                                                      | 40.00% (4/10)     | 40.00% (4/10)     | 33.33% (2/6)     | 50.00% (2/4)     |
| src/vault/AdminProxy.sol                                                            | 100.00% (1/1)     | 100.00% (1/1)     | 100.00% (0/0)    | 100.00% (1/1)    |
| src/vault/CloneFactory.sol                                                          | 100.00% (5/5)     | 100.00% (6/6)     | 100.00% (4/4)    | 100.00% (1/1)    |
| src/vault/CloneRegistry.sol                                                         | 66.67% (4/6)      | 66.67% (4/6)      | 100.00% (0/0)    | 33.33% (1/3)     |
| src/vault/DeploymentController.sol                                                  | 100.00% (13/13)   | 93.33% (14/15)    | 50.00% (1/2)     | 100.00% (6/6)    |
| src/vault/PermissionRegistry.sol                                                    | 100.00% (8/8)     | 83.33% (10/12)    | 50.00% (2/4)     | 100.00% (3/3)    |
| src/vault/TemplateRegistry.sol                                                      | 100.00% (18/18)   | 80.00% (16/20)    | 50.00% (4/8)     | 100.00% (6/6)    |
| src/vault/Vault.sol                                                                 | 84.17% (101/120)  | 79.31% (115/145)  | 57.14% (24/42)   | 74.29% (26/35)   |
| src/vault/VaultController.sol                                                       | 98.87% (175/177)  | 88.21% (247/280)  | 52.56% (41/78)   | 100.00% (42/42)  |
| src/vault/VaultRegistry.sol                                                         | 60.00% (6/10)     | 63.64% (7/11)     | 100.00% (2/2)    | 33.33% (2/6)     |
| src/vault/adapter/abstracts/AdapterBase.sol                                         | 73.40% (69/94)    | 68.75% (77/112)   | 41.67% (10/24)   | 39.47% (15/38)   |
| src/vault/adapter/abstracts/WithRewards.sol                                         | 0.00% (0/1)       | 0.00% (0/1)       | 100.00% (0/0)    | 0.00% (0/3)      |
| src/vault/adapter/beefy/BeefyAdapter.sol                                            | 0.00% (0/47)      | 0.00% (0/58)      | 0.00% (0/20)     | 0.00% (0/13)     |
| src/vault/adapter/yearn/YearnAdapter.sol                                            | 0.00% (0/28)      | 0.00% (0/36)      | 0.00% (0/8)      | 0.00% (0/13)     |
| src/vault/strategy/Pool2SingleAssetCompounder.sol                                   | 0.00% (0/28)      | 0.00% (0/47)      | 0.00% (0/4)      | 0.00% (0/3)      |
| src/vault/strategy/RewardsClaimer.sol                                               | 0.00% (0/9)       | 0.00% (0/15)      | 100.00% (0/0)    | 0.00% (0/1)      |
| src/vault/strategy/StrategyBase.sol                                                 | 0.00% (0/4)       | 0.00% (0/9)       | 0.00% (0/4)      | 0.00% (0/4)      |
| test/utils/EnhancedTest.sol                                                         | 0.00% (0/38)      | 0.00% (0/42)      | 0.00% (0/16)     | 0.00% (0/4)      |
| test/utils/Faucet.sol                                                               | 0.00% (0/27)      | 0.00% (0/29)      | 100.00% (0/0)    | 0.00% (0/8)      |
| test/utils/mocks/ClonableWithInitData.sol                                           | 100.00% (1/1)     | 100.00% (1/1)     | 100.00% (0/0)    | 100.00% (1/1)    |
| test/utils/mocks/ClonableWithoutInitData.sol                                        | 100.00% (2/2)     | 100.00% (2/2)     | 100.00% (0/0)    | 100.00% (2/2)    |
| test/utils/mocks/MockAdapter.sol                                                    | 100.00% (11/11)   | 100.00% (11/11)   | 100.00% (2/2)    | 100.00% (6/6)    |
| test/utils/mocks/MockERC20.sol                                                      | 66.67% (2/3)      | 66.67% (2/3)      | 100.00% (0/0)    | 66.67% (2/3)     |
| test/utils/mocks/MockERC4626.sol                                                    | 70.00% (28/40)    | 67.39% (31/46)    | 50.00% (4/8)     | 50.00% (9/18)    |
| test/utils/mocks/MockStrategy.sol                                                   | 100.00% (4/4)     | 100.00% (4/4)     | 100.00% (0/0)    | 100.00% (4/4)    |
| test/vault/integration/abstract/PropertyTest.prop.sol                               | 0.00% (0/86)      | 0.00% (0/126)     | 0.00% (0/8)      | 0.00% (0/16)     |
| test/vault/integration/beefy/BeefyTestConfigStorage.sol                             | 0.00% (0/2)       | 0.00% (0/2)       | 100.00% (0/0)    | 0.00% (0/2)      |
| test/vault/integration/beefy/BeefyVault.t.sol                                       | 0.00% (0/26)      | 0.00% (0/31)      | 100.00% (0/0)    | 0.00% (0/5)      |
| test/vault/integration/yearn/YearnTestConfigStorage.sol                             | 0.00% (0/2)       | 0.00% (0/2)       | 100.00% (0/0)    | 0.00% (0/2)      |
| test/vault/integration/yearn/YearnVault.t.sol                                       | 0.00% (0/14)      | 0.00% (0/16)      | 100.00% (0/0)    | 0.00% (0/5)      |
| Total                                                                               | 51.30% (747/1456) | 48.57% (884/1820) | 32.60% (163/500) | 50.23% (221/440) |

Due to its capacity, test coverage is expected to be 100%.

### [L-8] LACK OF INPUT VALIDATION

#### Description:

For defence-in-depth purpose, it is recommended to perform additional validation against the amount that the user is attempting to deposit, mint, withdraw and redeem to ensure that the submitted amount is valid.

OpenZeppelinTokenizedVault.sol#L9

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardStaking.sol

107:       function _deposit(
                address caller,
                address receiver,
                uint256 assets,
                uint256 shares
            ) internal override accrueRewards(caller, receiver) {
                IERC20(asset()).safeTransferFrom(caller, address(this), assets);

                _mint(receiver, shares);

                emit Deposit(caller, receiver, assets, shares);
            }


121:       function _withdraw(
                address caller,
                address receiver,
                address owner,
                uint256 assets,
                uint256 shares
            ) internal override accrueRewards(caller, receiver) {
                if (caller != owner) _approve(owner, msg.sender, allowance(owner, msg.sender) - shares);

                _burn(owner, shares);
                IERC20(asset()).safeTransfer(receiver, assets);

                emit Withdraw(caller, receiver, owner, assets, shares);
            }

137:     function _transfer(
                address from,
                address to,
                uint256 amount
            ) internal override accrueRewards(from, to) {
                if (from == address(0) || to == address(0)) revert ZeroAddressTransfer(from, to);

                uint256 fromBalance = balanceOf(from);
                if (fromBalance < amount) revert InsufficentBalance();

                _burn(from, amount);
                _mint(to, amount);

                emit Transfer(from, to, amount);
            }

```

```solidity
File: src/vault/Vault.sol

124:     function deposit(uint256 assets) public returns (uint256) {
            return deposit(assets, msg.sender);
     }

134:      function deposit(uint256 assets, address receiver)
            public
            nonReentrant
            whenNotPaused
            syncFeeCheckpoint
            returns (uint256 shares)
        {
            if (receiver == address(0)) revert InvalidReceiver();

            uint256 feeShares = convertToShares(
                assets.mulDiv(uint256(fees.deposit), 1e18, Math.Rounding.Down)
            );

            shares = convertToShares(assets) - feeShares;

            if (feeShares > 0) _mint(feeRecipient, feeShares);

            _mint(receiver, shares);

            asset.safeTransferFrom(msg.sender, address(this), assets);

            adapter.deposit(assets, address(this));

            emit Deposit(msg.sender, receiver, assets, shares);
        }

160:     function mint(uint256 shares) external returns (uint256) {
            return mint(shares, msg.sender);
    }

170:     function mint(uint256 shares, address receiver)
            public
            nonReentrant
            whenNotPaused
            syncFeeCheckpoint
            returns (uint256 assets)
        {
            if (receiver == address(0)) revert InvalidReceiver();

            uint256 depositFee = uint256(fees.deposit);

            uint256 feeShares = shares.mulDiv(
                depositFee,
                1e18 - depositFee,
                Math.Rounding.Down
            );

            assets = convertToAssets(shares + feeShares);

            if (feeShares > 0) _mint(feeRecipient, feeShares);

            _mint(receiver, shares);

            asset.safeTransferFrom(msg.sender, address(this), assets);

            adapter.deposit(assets, address(this));

            emit Deposit(msg.sender, receiver, assets, shares);
        }

200:     function withdraw(uint256 assets) public returns (uint256) {
            return withdraw(assets, msg.sender, msg.sender);
    }

242:     function redeem(uint256 shares) external returns (uint256) {
            return redeem(shares, msg.sender, msg.sender);
    }

```

### [L-9] LOSS OF PRECISION DUE TO ROUNDING

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardEscrow.sol
180:               Math.min(
                        (escrow.balance * (block.timestamp - uint256(escrow.lastUpdateTime))) /
                        (uint256(escrow.end) - uint256(escrow.lastUpdateTime)),
                        escrow.balance
                    );

```

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

122:                 ((lockedFundsRatio * lockedProfit) / DEGRADATION_COEFFICIENT);

```

### [L-10] IMPLMENTATION IS NOT FULLY UP TO EIP-4626’S SPECIFICATION

#### Description:

Must return the maximum amount of shares mint would allow to be deposited to receiver and not cause a revert, which must not be higher than the actual maximum that would be accepted (it should underestimate if necessary).

This assumes that the user has infinite assets, i.e. must not rely on` balanceOf` of asset.

Could cause unexpected behavior in the future due to non-compliance with EIP-4626 standard.

Similar problem for Sentimentxyz: https://github.com/sentimentxyz/protocol/pull/235/files

#### **Proof Of Concept**

```solidity
File: src/vault/Vault.sol

404:             function maxMint(address caller) external view returns (uint256) {
        return adapter.maxMint(caller);
    }

```

#### Recommended Mitigation Steps:

maxMint() and maxDeposit() should reflect the limitation of maxSupply.

### [L-11] MINE - MISSING CALLS TO \_\_REENTRANCYGUARD_INIT FUNCTIONS OF INHERITED CONTRACTS

#### Description:

Most contracts use the `delegateCall` proxy pattern and hence their implementations require the use of `initialize()` functions instead of constructors. This requires derived contracts to call the corresponding init functions of their inherited base contracts. This is done in most places except a few.

Impact: The inherited base classes do not get initialized which may lead to undefined behavior.

Missing call to `__ReentrancyGuard_init`:

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardStaking.sol

46:        __ERC4626_init(IERC20Metadata(address(_stakingToken)));
47:        __Owned_init(_owner);

```

```solidity
File: src/vault/Vault.sol

64:        __ERC20_init(
72:        __Owned_init(owner);

```

#### Recommended Mitigation Steps:

Add missing calls to init functions of inherited contracts.

### [L-12] SOLMATE’S SAFETRANSFERLIB DOESN’T CHECK WHETHER THE ERC20 CONTRACT EXISTS

#### Description:

Solmate’s SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn’t exist (yet).

This is stated in the Solmate library: https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardEscrow.sol

111:       token.safeTransfer(feeRecipient, fee);

165:       escrow.token.safeTransfer(escrow.account, claimable);

```

```solidity
File: src/utils/MultiRewardStaking.sol

131:     IERC20(asset()).safeTransfer(receiver, assets);

200:     rewardToken.safeTransfer(user, payout);

```

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

230:         IERC20(asset()).safeTransfer(receiver, assets);

```

#### Recommended Mitigation Steps:

Add a contract exist control in functions

### [L-13] Modifier side-effects

#### Description:

Modifiers should only implement checks and not make state changes and external calls which violates the [checks-effects-interactions](https://solidity.readthedocs.io/en/develop/security-considerations.html#use-the-checks-effects-interactions-pattern) pattern. These side-effects may go unnoticed by developers/auditors because the modifier code is typically far from the function implementation.

#### **Proof Of Concept**

```solidity
File: src/vault/Vault.sol

480:     modifier takeFees() {

496:     modifier syncFeeCheckpoint() {

```

```solidity
File: src/vault/VaultController.sol

704:   modifier canCreate() {

```

### [L-14] A SINGLE POINT OF FAILURE

#### Description:

The owner role has a single point of failure and onlyOwner can use critical a few functions.

Owner is not behind a multisig and changes are not behind a timelock.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

`onlyOwner` functions

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardEscrow.sol

207:   function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {

```

```solidity
File: src/utils/MultiRewardStaking.sol

251:   ) external onlyOwner {

296:   function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {

```

```solidity
File: src/vault/AdminProxy.sol

21:     onlyOwner

```

```solidity
File: src/vault/CloneFactory.sol

39:   function deploy(Template calldata template, bytes calldata data) external onlyOwner returns (address clone) {

```

```solidity
File: src/vault/CloneRegistry.sol

45:   ) external onlyOwner {

```

```solidity
File: src/vault/DeploymentController.sol

55:   function addTemplateCategory(bytes32 templateCategory) external onlyOwner {

80:   function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

103:   ) external onlyOwner returns (address clone) {

121:   function nominateNewDependencyOwner(address _owner) external onlyOwner {

```

```solidity
File: src/vault/PermissionRegistry.sol

38:   function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

```

```solidity
File: src/vault/TemplateRegistry.sol

52:   function addTemplateCategory(bytes32 templateCategory) external onlyOwner {

71:   ) external onlyOwner {

102:   function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

```

```solidity
File: src/vault/Vault.sol

525:     function proposeFees(VaultFees calldata newFees) external onlyOwner {

553:     function setFeeRecipient(address _feeRecipient) external onlyOwner {

578:     function proposeAdapter(IERC4626 newAdapter) external onlyOwner {

629:     function setQuitPeriod(uint256 _quitPeriod) external onlyOwner {

643:     function pause() external onlyOwner {

648:     function unpause() external onlyOwner {

```

```solidity
File: src/vault/VaultController.sol

408:   function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

543:   function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {

561:   function addTemplateCategories(bytes32[] calldata templateCategories) external onlyOwner {

581:     onlyOwner

723:   function nominateNewAdminProxyOwner(address newOwner) external onlyOwner {

751:   function setPerformanceFee(uint256 newFee) external onlyOwner {

764:   function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {

791:   function setHarvestCooldown(uint256 newCooldown) external onlyOwner {

804:   function setAdapterHarvestCooldowns(address[] calldata adapters) external onlyOwner {

832:   function setDeploymentController(IDeploymentController _deploymentController) external onlyOwner {

864:   function setActiveTemplateId(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

```

```solidity
File: src/vault/VaultRegistry.sol

44:   function registerVault(VaultMetadata calldata _metadata) external onlyOwner {

```

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

500:     function setHarvestCooldown(uint256 newCooldown) external onlyOwner {

549:     function setPerformanceFee(uint256 newFee) public onlyOwner {

574:     function pause() external onlyOwner {

582:     function unpause() external onlyOwner {

```

#### Recommended Mitigation Steps:

Add a time lock to critical functions. Admin-only functions that change critical parameters should emit events and have timelocks.

Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services.

Allow only multi-signature wallets to call the function to reduce the likelihood of an attack.

https://twitter.com/danielvf/status/1572963475101556738?s=20&t=V1kvzfJlsx-D2hfnG0OmuQ

Also detail them in documentation and NatSpec comments.

### [L-15] Block values as a proxy for time

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as `block.timestamp`, and `block.number` can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of `block.timestamp`, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers cant rely on the preciseness of the provided timestamp.

https://swcregistry.io/docs/SWC-116

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardEscrow.sol

114:     uint32 start = block.timestamp.safeCastTo32() + offset;

163:       escrows[escrowId].lastUpdateTime = block.timestamp.safeCastTo32();

175:       block.timestamp.safeCastTo32() < escrow.start

181:         (escrow.balance * (block.timestamp - uint256(escrow.lastUpdateTime))) /

```

```solidity
File: src/utils/MultiRewardStaking.sol

276:       ? block.timestamp.safeCastTo32()

284:       lastUpdatedTimestamp: block.timestamp.safeCastTo32()

309:       prevEndTime > block.timestamp ? prevEndTime : block.timestamp.safeCastTo32(),

309:       prevEndTime > block.timestamp ? prevEndTime : block.timestamp.safeCastTo32(),

346:     rewardInfos[rewardToken].lastUpdatedTimestamp = block.timestamp.safeCastTo32();

356:     if (rewardsEndTimestamp > block.timestamp)

357:       amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);

359:     return (block.timestamp + (amount / uint256(rewardsPerSecond))).safeCastTo32();

392:     if (rewards.rewardsEndTimestamp > block.timestamp) {

393:       elapsed = block.timestamp - rewards.lastUpdatedTimestamp;

409:     rewardInfos[_rewardToken].lastUpdatedTimestamp = block.timestamp.safeCastTo32();

454:     if (deadline < block.timestamp) revert PermitDeadlineExpired(deadline);

```

```solidity
File: src/vault/Vault.sol

87:         feesUpdatedAt = block.timestamp;

94:             abi.encodePacked("Popcorn", name(), block.timestamp, "Vault")

434:                     totalAssets() * (block.timestamp - feesUpdatedAt),

488:         if (managementFee > 0) feesUpdatedAt = block.timestamp;

534:         proposedFeeTime = block.timestamp;

536:         emit NewFeesProposed(newFees, block.timestamp);

541:         if (block.timestamp < proposedFeeTime + quitPeriod)

583:         proposedAdapterTime = block.timestamp;

585:         emit NewAdapterProposed(newAdapter, block.timestamp);

595:         if (block.timestamp < proposedAdapterTime + quitPeriod)

673:         if (deadline < block.timestamp) revert PermitDeadlineExpired(deadline);

```

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

86:         lastHarvest = block.timestamp;

441:             ((lastHarvest + harvestCooldown) < block.timestamp)

641:         if (deadline < block.timestamp) revert PermitDeadlineExpired(deadline);

```

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

115:         uint256 lockedFundsRatio = (block.timestamp - yVault.lastReport()) *

```

### [L-16] Account existence check for low-level calls

#### Description:

Low-level calls `call`/`delegatecall`/`staticcall` return true even if the account called is non-existent (per EVM design). Account existence must be checked prior to calling if needed.

https://github.com/crytic/slither/wiki/Detector-Documentation#low-level-callsn

#### **Proof Of Concept**

```solidity
File: src/vault/AdminProxy.sol

24:     return target.call(callData);

```

```solidity
File: src/vault/CloneFactory.sol

43:     if (template.requiresInitData) (success, ) = clone.call(data);

```

### [L-17] UNSAFE CAST

#### Description:

Keep in mind that the version of solidity used, despite being greater than 0.8, does not prevent integer overflows during casting, it only does so in mathematical operations.

It is necessary to safely convert between the different numeric types.

#### **Proof Of Concept**

```solidity
File: src/vault/VaultController.sol

314:     uint8 len = uint8(vaults.length);

336:     uint8 len = uint8(vaults.length);

353:     uint8 len = uint8(vaults.length);

373:     uint8 len = uint8(vaults.length);

436:     uint8 len = uint8(vaults.length);

488:     uint8 len = uint8(vaults.length);

517:     uint8 len = uint8(vaults.length);

563:     uint8 len = uint8(templateCategories.length);

583:     uint8 len = uint8(templateCategories.length);

606:     uint8 len = uint8(vaults.length);

619:     uint8 len = uint8(vaults.length);

632:     uint8 len = uint8(vaults.length);

645:     uint8 len = uint8(vaults.length);

765:     uint8 len = uint8(adapters.length);

805:     uint8 len = uint8(adapters.length);

```

#### Recommended Mitigation Steps:

Use a safeCast from Open Zeppelin or increase the type length.

### [L-18] NOT USING THE LATEST VERSION OF OPENZEPPELIN FROM DEPENDENCIES

#### Description:

The package.json configuration file says that the project is using 4.8.0 and 4.7.1 of OpenZeppelin which has a not last update version.

#### **Proof Of Concept**

```solidity
File: package.json
    "@openzeppelin/contracts": "4.8.0",
    "openzeppelin-upgradeable": "npm:@openzeppelin/contracts-upgradeable@4.7.1",
```

#### Recommended Mitigation Steps:

Use patched versions. Latest non vulnerable version 4.8.1

### [L-19] Unspecific compiler version pragma

#### **Context:**

All Contracts
