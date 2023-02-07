# Gas Report 

[Exlusive findings](#exlusive-findings) is below the [automated findings](#automated-findings).

---

## Automated Findings
## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use assembly to check for `address(0)` | 24 |
| [GAS-2](#GAS-2) | Using bools for storage incurs overhead | 3 |
| [GAS-3](#GAS-3) | Cache array length outside of loop | 5 |
| [GAS-4](#GAS-4) | State variables should be cached in stack variables rather than re-reading them from storage | 3 |
| [GAS-5](#GAS-5) | Use calldata instead of memory for function arguments that do not get mutated | 16 |
| [GAS-6](#GAS-6) | Don't initialize variables with default value | 19 |
| [GAS-7](#GAS-7) | Functions guaranteed to revert when called by normal users can be marked `payable` | 28 |
| [GAS-8](#GAS-8) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 24 |
| [GAS-9](#GAS-9) | Using `private` rather than `public` for constants, saves gas | 1 |
| [GAS-10](#GAS-10) | Use != 0 instead of > 0 for unsigned integer comparison | 23 |
### <a name="GAS-1"></a>[GAS-1] Use assembly to check for `address(0)`
*Saves 6 gas per instance*

*Instances (24)*:
```solidity
File: src/utils/MultiRewardEscrow.sol

96:     if (account == address(0)) revert ZeroAddress();

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol)

```solidity
File: src/utils/MultiRewardStaking.sol

142:     if (from == address(0) || to == address(0)) revert ZeroAddressTransfer(from, to);

142:     if (from == address(0) || to == address(0)) revert ZeroAddressTransfer(from, to);

481:       if (recoveredAddress == address(0) || recoveredAddress != owner) revert InvalidSigner(recoveredAddress);

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol)

```solidity
File: src/vault/Vault.sol

74:         if (address(asset_) == address(0)) revert InvalidAsset();

90:         if (feeRecipient_ == address(0)) revert InvalidFeeRecipient();

141:         if (receiver == address(0)) revert InvalidReceiver();

177:         if (receiver == address(0)) revert InvalidReceiver();

216:         if (receiver == address(0)) revert InvalidReceiver();

258:         if (receiver == address(0)) revert InvalidReceiver();

554:         if (_feeRecipient == address(0)) revert InvalidFeeRecipient();

702:             if (recoveredAddress == address(0) || recoveredAddress != owner)

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol)

```solidity
File: src/vault/VaultController.sol

108:     if (staking == address(0)) staking = _deployStaking(IERC20(address(vault)), _deploymentController);

687:       token == address(0)

693:     if (adapter != address(0)) {

837:     if (address(_deploymentController) == address(0) || address(deploymentController) == address(_deploymentController))

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol)

```solidity
File: src/vault/VaultRegistry.sol

45:     if (metadata[_metadata.vault].vault != address(0)) revert VaultAlreadyRegistered();

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol)

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

62:             _beefyBooster != address(0) &&

77:             _beefyBooster == address(0) ? _beefyVault : _beefyBooster

82:         if (_beefyBooster != address(0))

138:         if (address(beefyBooster) == address(0)) return _rewardTokens;

198:         if (address(beefyBooster) != address(0))

209:         if (address(beefyBooster) != address(0))

222:         if (address(beefyBooster) == address(0)) revert NoBeefyBooster();

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol)

### <a name="GAS-2"></a>[GAS-2] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (3)*:
```solidity
File: src/vault/CloneRegistry.sol

28:   mapping(address => bool) public cloneExists;

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol)

```solidity
File: src/vault/TemplateRegistry.sol

33:   mapping(bytes32 => bool) public templateExists;

35:   mapping(bytes32 => bool) public templateCategoryExists;

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol)

### <a name="GAS-3"></a>[GAS-3] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (5)*:
```solidity
File: src/utils/MultiRewardEscrow.sol

53:     for (uint256 i = 0; i < escrowIds.length; i++) {

155:     for (uint256 i = 0; i < escrowIds.length; i++) {

210:     for (uint256 i = 0; i < tokens.length; i++) {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol)

```solidity
File: src/utils/MultiRewardStaking.sol

171:     for (uint8 i; i < _rewardTokens.length; i++) {

373:     for (uint8 i; i < _rewardTokens.length; i++) {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol)

### <a name="GAS-4"></a>[GAS-4] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*Saves 100 gas per instance*

*Instances (3)*:
```solidity
File: src/vault/Vault.sol

545:         fees = proposedFees;

606:         emit ChangedAdapter(adapter, proposedAdapter);

608:         adapter = proposedAdapter;

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol)

### <a name="GAS-5"></a>[GAS-5] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

*Instances (16)*:
```solidity
File: src/utils/MultiRewardEscrow.sol

154:   function claimRewards(bytes32[] memory escrowIds) external {

207:   function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {

207:   function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol)

```solidity
File: src/utils/MultiRewardStaking.sol

170:   function claimRewards(address user, IERC20[] memory _rewardTokens) external accrueRewards(msg.sender, user) {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol)

```solidity
File: src/vault/VaultController.sol

91:     DeploymentArgs memory adapterData,

92:     DeploymentArgs memory strategyData,

94:     bytes memory rewardsData,

95:     VaultMetadata memory metadata,

188:     DeploymentArgs memory adapterData,

189:     DeploymentArgs memory strategyData,

433:   function addStakingRewardsTokens(address[] memory vaults, bytes[] memory rewardTokenData) public {

433:   function addStakingRewardsTokens(address[] memory vaults, bytes[] memory rewardTokenData) public {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol)

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

47:         bytes memory adapterInitData,

49:         bytes memory beefyInitData

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol)

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

35:         bytes memory adapterInitData,

37:         bytes memory

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol)

### <a name="GAS-6"></a>[GAS-6] Don't initialize variables with default value

*Instances (19)*:
```solidity
File: src/utils/MultiRewardEscrow.sol

53:     for (uint256 i = 0; i < escrowIds.length; i++) {

155:     for (uint256 i = 0; i < escrowIds.length; i++) {

210:     for (uint256 i = 0; i < tokens.length; i++) {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol)

```solidity
File: src/vault/PermissionRegistry.sol

42:     for (uint256 i = 0; i < len; i++) {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol)

```solidity
File: src/vault/VaultController.sol

319:     for (uint8 i = 0; i < len; i++) {

337:     for (uint8 i = 0; i < len; i++) {

357:     for (uint8 i = 0; i < len; i++) {

374:     for (uint8 i = 0; i < len; i++) {

437:     for (uint256 i = 0; i < len; i++) {

494:     for (uint256 i = 0; i < len; i++) {

523:     for (uint256 i = 0; i < len; i++) {

564:     for (uint256 i = 0; i < len; i++) {

587:     for (uint256 i = 0; i < len; i++) {

607:     for (uint256 i = 0; i < len; i++) {

620:     for (uint256 i = 0; i < len; i++) {

633:     for (uint256 i = 0; i < len; i++) {

646:     for (uint256 i = 0; i < len; i++) {

766:     for (uint256 i = 0; i < len; i++) {

806:     for (uint256 i = 0; i < len; i++) {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol)

### <a name="GAS-7"></a>[GAS-7] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (28)*:
```solidity
File: src/utils/MultiRewardEscrow.sol

207:   function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol)

```solidity
File: src/utils/MultiRewardStaking.sol

296:   function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol)

```solidity
File: src/vault/CloneFactory.sol

39:   function deploy(Template calldata template, bytes calldata data) external onlyOwner returns (address clone) {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneFactory.sol)

```solidity
File: src/vault/DeploymentController.sol

55:   function addTemplateCategory(bytes32 templateCategory) external onlyOwner {

80:   function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

121:   function nominateNewDependencyOwner(address _owner) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol)

```solidity
File: src/vault/PermissionRegistry.sol

38:   function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol)

```solidity
File: src/vault/TemplateRegistry.sol

52:   function addTemplateCategory(bytes32 templateCategory) external onlyOwner {

102:   function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol)

```solidity
File: src/vault/Vault.sol

525:     function proposeFees(VaultFees calldata newFees) external onlyOwner {

553:     function setFeeRecipient(address _feeRecipient) external onlyOwner {

578:     function proposeAdapter(IERC4626 newAdapter) external onlyOwner {

629:     function setQuitPeriod(uint256 _quitPeriod) external onlyOwner {

643:     function pause() external onlyOwner {

648:     function unpause() external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol)

```solidity
File: src/vault/VaultController.sol

408:   function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

543:   function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {

561:   function addTemplateCategories(bytes32[] calldata templateCategories) external onlyOwner {

723:   function nominateNewAdminProxyOwner(address newOwner) external onlyOwner {

751:   function setPerformanceFee(uint256 newFee) external onlyOwner {

764:   function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {

791:   function setHarvestCooldown(uint256 newCooldown) external onlyOwner {

804:   function setAdapterHarvestCooldowns(address[] calldata adapters) external onlyOwner {

832:   function setDeploymentController(IDeploymentController _deploymentController) external onlyOwner {

864:   function setActiveTemplateId(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol)

```solidity
File: src/vault/VaultRegistry.sol

44:   function registerVault(VaultMetadata calldata _metadata) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol)

```solidity
File: src/vault/adapter/abstracts/WithRewards.sol

15:   function claim() public virtual onlyStrategy {}

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/WithRewards.sol)

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

221:     function claim() public override onlyStrategy {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol)

### <a name="GAS-8"></a>[GAS-8] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
*Saves 5 gas per loop*

*Instances (24)*:
```solidity
File: src/utils/MultiRewardEscrow.sol

53:     for (uint256 i = 0; i < escrowIds.length; i++) {

102:     nonce++;

155:     for (uint256 i = 0; i < escrowIds.length; i++) {

210:     for (uint256 i = 0; i < tokens.length; i++) {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol)

```solidity
File: src/utils/MultiRewardStaking.sol

171:     for (uint8 i; i < _rewardTokens.length; i++) {

373:     for (uint8 i; i < _rewardTokens.length; i++) {

470:                 nonces[owner]++,

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol)

```solidity
File: src/vault/PermissionRegistry.sol

42:     for (uint256 i = 0; i < len; i++) {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol)

```solidity
File: src/vault/Vault.sol

691:                                 nonces[owner]++,

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol)

```solidity
File: src/vault/VaultController.sol

319:     for (uint8 i = 0; i < len; i++) {

337:     for (uint8 i = 0; i < len; i++) {

357:     for (uint8 i = 0; i < len; i++) {

374:     for (uint8 i = 0; i < len; i++) {

437:     for (uint256 i = 0; i < len; i++) {

494:     for (uint256 i = 0; i < len; i++) {

523:     for (uint256 i = 0; i < len; i++) {

564:     for (uint256 i = 0; i < len; i++) {

587:     for (uint256 i = 0; i < len; i++) {

607:     for (uint256 i = 0; i < len; i++) {

620:     for (uint256 i = 0; i < len; i++) {

633:     for (uint256 i = 0; i < len; i++) {

646:     for (uint256 i = 0; i < len; i++) {

766:     for (uint256 i = 0; i < len; i++) {

806:     for (uint256 i = 0; i < len; i++) {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol)

### <a name="GAS-9"></a>[GAS-9] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (1)*:
```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

31:     uint256 public constant BPS_DENOMINATOR = 10_000;

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol)

### <a name="GAS-10"></a>[GAS-10] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (23)*:
```solidity
File: src/utils/MultiRewardEscrow.sol

107:     if (feePerc > 0) {

141:     return escrows[escrowId].lastUpdateTime != 0 && escrows[escrowId].balance > 0;

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol)

```solidity
File: src/utils/MultiRewardStaking.sol

178:       if (escrowInfo.escrowPercentage > 0) {

255:     if (rewards.lastUpdatedTimestamp > 0) revert RewardTokenAlreadyExist(rewardToken);

257:     if (amount > 0) {

341:     if (rewards.rewardsPerSecond > 0) {

377:       if (rewards.rewardsPerSecond > 0) _accrueRewards(rewardToken, _accrueStatic(rewards));

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol)

```solidity
File: src/vault/Vault.sol

149:         if (feeShares > 0) _mint(feeRecipient, feeShares);

189:         if (feeShares > 0) _mint(feeRecipient, feeShares);

235:         if (feeShares > 0) _mint(feeRecipient, feeShares);

273:         if (feeShares > 0) _mint(feeRecipient, feeShares);

432:             managementFee > 0

453:             performanceFee > 0 && shareValue > highWaterMark

488:         if (managementFee > 0) feesUpdatedAt = block.timestamp;

490:         if (totalFee > 0 && currentAssets > 0)

490:         if (totalFee > 0 && currentAssets > 0)

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol)

```solidity
File: src/vault/VaultController.sol

103:     if (adapterData.id > 0)

112:     if (rewardsData.length > 0) _handleVaultStakingRewards(vault, rewardsData);

169:     if (initialDeposit > 0) {

211:     if (strategyData.id > 0) {

694:       if (adapterId > 0) revert AdapterConfigFaulty();

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol)

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

156:         if (beefyFee > 0)

177:         if (beefyFee > 0)

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol)


---
## Exlusive Findings
---
### Gas Optimizations Report
| No. | Issue |
| --- | --- |
| 1 | [Using `uint256` instead of `bool` in mappings is more gas efficient [146 gas per instance]](#G-01-using-uint256-instead-of-bool-in-mappings-is-more-gas-efficient-146-gas-per-instance)|
| 2 | [Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead](#G-02-usage-of-uintint-smaller-than-32-bytes-256-bits-incurs-overhead)|
| 3 | [++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for-loop and while-loops](#G-03-ii-should-be-uncheckediuncheckedi-when-it-is-not-possible-for-them-to-overflow-as-is-the-case-when-used-in-for-loop-and-while-loops)|
| 4 | [Optimize names to save gas [22 gas per instance]](#G-04-optimize-names-to-save-gas-22-gas-per-instance)|
| 5 | [Setting the `constructor` to `payable` [~13 gas per instance]](#G-05-setting-the-constructor-to-payable-13-gas-per-instance)|
| 6 | [Superfluous event fields](#G-06-superfluous-event-fields)|
| 7 | [Comparison operators](#G-07-comparison-operators)|
| 8 | [Use a more recent version of solidity](#G-08-use-a-more-recent-version-of-solidity)|
| 9 | [`<array>.length` should not be looked up in every loop of a for-loop](#G-09-arraylength-should-not-be-looked-up-in-every-loop-of-a-for-loop)|
| 10 | [`x += y` costs more gas than `x = x + y` for state variables](#G-10-x--y-costs-more-gas-than-x--x--y-for-state-variables)|
---
### [G-01] Using `uint256` instead of `bool` in mappings is more gas efficient [146 gas per instance]

---
**Description:**

OpenZeppelin uint256 and bool comparison: 

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled. The values being non-zero value makes deployment a bit more expensive.

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.

**Recommendation:**

Use uint256(1) and uint256(2) instead of bool.

**Lines of Code:** 

- [CloneRegistry.sol#L28](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol#L28)
- [TemplateRegistry.sol#L33](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol#L33)
- [TemplateRegistry.sol#L35](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol#L35)

---
### [G-02] Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

---
**Description:**

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so.

**Recommendation:**

Use a larger size then downcast where needed

**Lines of Code:** 

- [IMultiRewardEscrow.sol#L11](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol#L11)
- [IMultiRewardEscrow.sol#L13](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol#L13)
- [IMultiRewardEscrow.sol#L15](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol#L15)
- [IMultiRewardEscrow.sol#L36](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol#L36)
- [IMultiRewardEscrow.sol#L37](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol#L37)
- [IMultiRewardStaking.sol#L16](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L16)
- [IMultiRewardStaking.sol#L18](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L18)
- [IMultiRewardStaking.sol#L21](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L21)
- [IMultiRewardStaking.sol#L23](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L23)
- [IMultiRewardStaking.sol#L25](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L25)
- [IMultiRewardStaking.sol#L30](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L30)
- [IMultiRewardStaking.sol#L32](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L32)
- [IMultiRewardStaking.sol#L34](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L34)
- [IMultiRewardStaking.sol#L40](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L40)
- [IMultiRewardStaking.sol#L43](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L43)
- [IMultiRewardStaking.sol#L44](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L44)
- [IMultiRewardStaking.sol#L45](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L45)
- [IMultiRewardStaking.sol#L48](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L48)
- [IVault.sol#L9](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol#L9)
- [IVault.sol#L10](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol#L10)
- [IVault.sol#L11](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol#L11)
- [IVault.sol#L12](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol#L12)
- [MultiRewardEscrow.sol#L73](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L73)
- [MultiRewardEscrow.sol#L73](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L73)
- [MultiRewardEscrow.sol#L92](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L92)
- [MultiRewardEscrow.sol#L93](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L93)
- [MultiRewardEscrow.sol#L114](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L114)
- [MultiRewardStaking.sol#L33](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L33)
- [MultiRewardStaking.sol#L171](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L171)
- [MultiRewardStaking.sol#L220](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L220)
- [MultiRewardStaking.sol#L220](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L220)
- [MultiRewardStaking.sol#L245](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L245)
- [MultiRewardStaking.sol#L248](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L248)
- [MultiRewardStaking.sol#L249](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L249)
- [MultiRewardStaking.sol#L250](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L250)
- [MultiRewardStaking.sol#L274](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L274)
- [MultiRewardStaking.sol#L275](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L275)
- [MultiRewardStaking.sol#L296](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L296)
- [MultiRewardStaking.sol#L307](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L307)
- [MultiRewardStaking.sol#L308](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L308)
- [MultiRewardStaking.sol#L340](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L340)
- [MultiRewardStaking.sol#L352](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L352)
- [MultiRewardStaking.sol#L353](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L353)
- [MultiRewardStaking.sol#L373](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L373)
- [MultiRewardStaking.sol#L404](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L404)
- [MultiRewardStaking.sol#L450](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L450)
- [Vault.sol#L38](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L38)
- [Vault.sol#L669](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L669)
- [VaultController.sol#L314](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L314)
- [VaultController.sol#L319](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L319)
- [VaultController.sol#L336](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L336)
- [VaultController.sol#L337](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L337)
- [VaultController.sol#L353](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L353)
- [VaultController.sol#L357](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L357)
- [VaultController.sol#L373](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L373)
- [VaultController.sol#L374](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L374)
- [VaultController.sol#L436](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L436)
- [VaultController.sol#L440](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L440)
- [VaultController.sol#L443](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L443)
- [VaultController.sol#L444](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L444)
- [VaultController.sol#L488](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L488)
- [VaultController.sol#L517](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L517)
- [VaultController.sol#L563](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L563)
- [VaultController.sol#L583](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L583)
- [VaultController.sol#L606](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L606)
- [VaultController.sol#L619](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L619)
- [VaultController.sol#L632](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L632)
- [VaultController.sol#L645](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L645)
- [VaultController.sol#L765](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L765)
- [VaultController.sol#L805](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L805)
- [AdapterBase.sol#L38](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L38)
- [AdapterBase.sol#L637](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L637)

---
### [G-03] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for-loop and while-loops

---
**Description:**

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

**Recommendation:**

Using Solidity's unchecked block saves the overflow checks.

**Proof Of Concept:**

<https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g011---unnecessary-checked-arithmetic-in-for-loop>

**Lines of Code:** 

- [MultiRewardEscrow.sol#L53](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L53)
- [MultiRewardEscrow.sol#L155](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L155)
- [MultiRewardEscrow.sol#L210](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L210)
- [MultiRewardStaking.sol#L171](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L171)
- [MultiRewardStaking.sol#L373](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L373)
- [PermissionRegistry.sol#L42](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol#L42)
- [VaultController.sol#L319](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L319)
- [VaultController.sol#L337](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L337)
- [VaultController.sol#L357](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L357)
- [VaultController.sol#L374](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L374)
- [VaultController.sol#L437](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L437)
- [VaultController.sol#L494](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L494)
- [VaultController.sol#L523](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L523)
- [VaultController.sol#L564](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L564)
- [VaultController.sol#L587](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L587)
- [VaultController.sol#L607](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L607)
- [VaultController.sol#L620](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L620)
- [VaultController.sol#L633](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L633)
- [VaultController.sol#L646](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L646)
- [VaultController.sol#L766](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L766)
- [VaultController.sol#L806](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L806)

---
### [G-04] Optimize names to save gas [22 gas per instance]

---
**Description:**

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

**Recommendation:**

Find a lower `method ID` name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas. For example, the function IDs in the L1GraphTokenGateway.sol contract will be the most used; A lower method ID may be given.

**Proof Of Concept:**

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

**Lines of Code:** 

- [IEIP165.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IEIP165.sol)
- [IMultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol)
- [IMultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol)
- [IAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IAdapter.sol)
- [IAdminProxy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IAdminProxy.sol)
- [ICloneFactory.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ICloneFactory.sol)
- [ICloneRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ICloneRegistry.sol)
- [IDeploymentController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IDeploymentController.sol)
- [IERC4626.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IERC4626.sol)
- [IPermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IPermissionRegistry.sol)
- [IStrategy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IStrategy.sol)
- [ITemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ITemplateRegistry.sol)
- [IVault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol)
- [IVaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVaultController.sol)
- [IVaultRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVaultRegistry.sol)
- [IWithRewards.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IWithRewards.sol)
- [EIP165.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/EIP165.sol)
- [MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol)
- [MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol)
- [AdminProxy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/AdminProxy.sol)
- [CloneFactory.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneFactory.sol)
- [CloneRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol)
- [DeploymentController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol)
- [PermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol)
- [TemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol)
- [Vault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol)
- [VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol)
- [VaultRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol)
- [AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol)
- [OnlyStrategy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/OnlyStrategy.sol)
- [WithRewards.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/WithRewards.sol)
- [BeefyAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol)
- [IBeefy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/IBeefy.sol)
- [IYearn.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/IYearn.sol)
- [YearnAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol)

---
### [G-05] Setting the `constructor` to `payable` [~13 gas per instance]

---
**Lines of Code:** 

- [MultiRewardEscrow.sol#L30](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L30)
- [AdminProxy.sol#L16](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/AdminProxy.sol#L16)
- [CloneFactory.sol#L23](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneFactory.sol#L23)
- [CloneRegistry.sol#L22](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol#L22)
- [DeploymentController.sol#L35](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol#L35)
- [PermissionRegistry.sol#L20](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol#L20)
- [TemplateRegistry.sol#L24](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol#L24)
- [VaultController.sol#L53](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L53)
- [VaultRegistry.sol#L21](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol#L21)

---
### [G-06] Superfluous event fields

---
**Description:**

`block.number` and `block.timestamp` are added to the event information by default, so adding them manually will waste additional gas.

**Lines of Code:** 

- [Vault.sol#L512](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L512)
- [Vault.sol#L569](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L569)

---
### [G-07] Comparison operators

---
**Description:**

In the EVM, there is no opcode for `>=` or `<=`. When using greater than or equal, two operations are performed: `>` and `=`. Using strict comparison operators hence saves gas.

**Recommendation:**

 Replace `<=` with `<`, and `>=` with `>`. Do not forget to increment/decrement the compared variable.

**Lines of Code:** 

- [MultiRewardEscrow.sol#L211](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L211)
- [Vault.sol#L527](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L527)
- [Vault.sol#L528](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L528)
- [Vault.sol#L529](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L529)
- [Vault.sol#L530](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L530)
- [AdapterBase.sol#L502](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L502)
- [YearnAdapter.sol#L150](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L150)

---
### [G-08] Use a more recent version of solidity

---
**Description:**

Solidity 0.8.10 has a useful change that reduced gas costs of external calls which expect a return value.

In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

**Lines of Code:** 

- [IEIP165.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IEIP165.sol)
- [IMultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol)
- [IMultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol)
- [IAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IAdapter.sol)
- [IAdminProxy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IAdminProxy.sol)
- [ICloneFactory.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ICloneFactory.sol)
- [ICloneRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ICloneRegistry.sol)
- [IDeploymentController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IDeploymentController.sol)
- [IERC4626.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IERC4626.sol)
- [IPermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IPermissionRegistry.sol)
- [IStrategy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IStrategy.sol)
- [ITemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ITemplateRegistry.sol)
- [IVault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol)
- [IVaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVaultController.sol)
- [IVaultRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVaultRegistry.sol)
- [IWithRewards.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IWithRewards.sol)
- [EIP165.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/EIP165.sol)
- [MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol)
- [MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol)
- [AdminProxy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/AdminProxy.sol)
- [CloneFactory.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneFactory.sol)
- [CloneRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol)
- [DeploymentController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol)
- [PermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol)
- [TemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol)
- [Vault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol)
- [VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol)
- [VaultRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol)
- [AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol)
- [OnlyStrategy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/OnlyStrategy.sol)
- [WithRewards.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/WithRewards.sol)
- [BeefyAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol)
- [IBeefy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/IBeefy.sol)
- [IYearn.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/IYearn.sol)
- [YearnAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol)

---
### [G-09] `<array>.length` should not be looked up in every loop of a for-loop

---
**Description:**

The overheads outlined below are *PER LOOP*, excluding the first loop 

- storage arrays incur a Gwarmaccess (100 gas)

- memory arrays use `MLOAD` (3 gas)

- calldata arrays use `CALLDATALOAD` (3 gas)

Caching the length changes each of these to a `DUP<N>` (3 gas), and gets rid of the extra DUP<N> needed to store the stack offset



**Lines of Code:** 

- [MultiRewardEscrow.sol#L53](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L53)
- [MultiRewardEscrow.sol#L155](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L155)
- [MultiRewardEscrow.sol#L210](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L210)
- [MultiRewardStaking.sol#L171](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L171)
- [MultiRewardStaking.sol#L373](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L373)

---
### [G-10] `x += y` costs more gas than `x = x + y` for state variables

---
**Lines of Code:** 

- [MultiRewardEscrow.sol#L162](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L162)
- [MultiRewardStaking.sol#L408](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L408)
- [MultiRewardStaking.sol#L431](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L431)