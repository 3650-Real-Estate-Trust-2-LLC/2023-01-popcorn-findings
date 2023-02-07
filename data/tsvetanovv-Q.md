## Low Risk Issues Summary
| Number |Issues Details
|:--:|:-------|:--:|
|[L-01]| Missing event for critical parameter change
|[L-02]| Loss of precision due to rounding
|[L-03]| Missing checks for address(0x0)
|[L-04]| Misleading constant name
***

## [L-01] MISSING EVENT FOR CRITICAL PARAMETER CHANGE

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.
```
src/vault/DeploymentController.sol
55: function addTemplateCategory(bytes32 templateCategory) external onlyOwner {
66: function addTemplate(
80: function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {
99: function deploy(
121: function nominateNewDependencyOwner(address _owner) external onlyOwner {
131: function acceptDependencyOwnership() external {

src/vault/TemplateRegistry.sol
115: function getTemplateCategories() external view returns (bytes32[] memory) {
119: function getTemplateIds(bytes32 templateCategory) external view returns (bytes32[] memory) {
123: function getTemplate(bytes32 templateCategory, bytes32 templateId) external view returns (Template memory) {
```
***
## [L-02] LOSS OF PRECISION DUE TO ROUNDING
```
src/vault/adapter/yearn/YearnAdapter.sol
122: ((lockedFundsRatio * lockedProfit) / DEGRADATION_COEFFICIENT);
```
***

## [L-03] MISSING CHECKS FOR ADDRESS(0X0)

Lack of zero-address validation on address parameters may lead to transaction reverts, waste gas, require resubmission of transactions and may even force contract redeployments in certain cases within the protocol.

```
VaultController.sol
141: function _registerCreatedVault(
142:    address vault,
143:    address staking,
```

#### Recommended Mitigation Steps

Consider adding explicit zero-address validation on input parameters of address type.
***

## [L-04] Misleading constant name
In `src/vault/Vault.sol` we have:
```
35: uint256 constant SECONDS_PER_YEAR = 365.25 days
```
But the name `SECONDS_PER_YEAR` is wrong. It should be `DAYS_PER_YEAR`. This mistaken name can lead to very bad consequences if the developer is trusted by name alone.
### Recommended Mitigation Steps
Change `SECONDS_PER_YEAR` to `DAYS_PER_YEAR`. Or keep `SECONDS_PER_YEAR` and change `365.25 days`  depending on what you need.


***
***


## Non-Critical Issues Summary
| Number |Issues Details
|:--:|:-------|:--:|
|[NC-01]| Use latest Solidity version
|[NC-02]| Use stable pragma statement
|[NC-03]| Update external dependency to latest version
|[NC-04]| Open `TODO` in the code
|[NC-05]| Events-emit should be at the end of the function
|[NC-06]| Add NatSpec documentation
|[NC-07]| Use scientific notation (E.G. 1e18) rather than exponentiation (E.G. 10**18)
|[NC-08]| Avoid using low call function ecrecover
|[NC-09]| NatSpec is incomplete
|[NC-10]| You can use named parameters in mapping types
***
## [NC-01] Use latest Solidity version

Solidity pragma versioning should be upgraded to latest available version. 
***

## [NC-02] Use stable pragma statement

Using a floating pragma statement `pragma solidity ^0.8.15;` is discouraged as code can compile to different bytecodes with different compiler versions. Use a stable pragma statement to get a deterministic bytecode.
***

## [NC-03] Update external dependency to latest version

The OpenZeppelin library dependency is using a stale version - upgrade to latest one to get all security patches, features and gas optimisations. The latest version is `4.8.1`

```
package.json:
25:     "@openzeppelin/contracts": "4.8.0"
26:     "openzeppelin-upgradeable": "npm:@openzeppelin/contracts-upgradeable@4.7.1"
```
***

## [NC-04] Open `TODO` in the code
The code that contains “open todos” reflects that the development is not finished and that the code can change a posteriori, prior release, with or without audit.

```
AdapterBase.sol:
516: // TODO use deterministic fee recipient proxy
```
***

## [NC-05] Events-emit should be at the end of the function
```
src/vault/PermissionRegistry.sol
45: emit PermissionSet(targets[i], newPermissions[i].endorsed, newPermissions[i].rejected);

src/vault/Vault.sol:
556: emit FeeRecipientUpdated(feeRecipient, _feeRecipient);
606: emit ChangedAdapter(adapter, proposedAdapter);

src/vault/VaultController.sol
755: emit PerformanceFeeChanged(performanceFee, newFee);
795: emit HarvestCooldownChanged(harvestCooldown, newCooldown);
```
***

## [NC-06] Add NatSpec documentation
NatSpec documentation to all public methods and variables is essential for better understanding of the code by developers and auditors and is strongly recommended.
NatSpec is missing for the following function:

```solidity
src/vault/CloneRegistry.sol
57: function getClonesByCategoryAndId(bytes32 templateCategory, bytes32 templateId)
65: function getAllClones() external view returns (address[] memory) {

src/vault/VaultRegistry.sol
59: function getVault(address vault) external view returns (VaultMetadata memory) {
63: function getVaultsByAsset(address asset) external view returns (address[] memory) {
67: function getTotalVaults() external view returns (uint256) {
71: function getRegisteredAddresses() external view returns (address[] memory) {
75: function getSubmitter(address vault) external view returns (VaultMetadata memory) {

src/vault/PermissionRegistry.sol
51: function endorsed(address target) external view returns (bool) {
55: function rejected(address target) external view returns (bool) {

src/vault/adapter/yearn/YearnAdapter.sol
57: function name()
66: function symbol()
84: function _underlyingBalance() internal view override returns (uint256) {
158: function _protocolDeposit(uint256 amount, uint256)

src/utils/MultiRewardEscrow.sol
38: function getEscrowIdsByUser(address account) external view returns (bytes32[] memory) {
42: function getEscrowIdsByUserAndToken(address account, IERC20 token) external view returns (bytes32[] memory) {
140: function isClaimable(bytes32 escrowId) external view returns (bool) {
144: function getClaimableAmount(bytes32 escrowId) external view returns (uint256) {

src/vault/adapter/beefy/BeefyAdapter.sol
86: function name()
95: function symbol()
108: function _totalAssets() internal view override returns (uint256) {
117: function _underlyingBalance() internal view override returns (uint256) {
230: function supportsInterface(bytes4 interfaceId)

src/utils/MultiRewardStaking.sol 
351: function _calcRewardsEnd(
362: function getAllRewardsTokens() external view returns (IERC20[] memory) {
445: function permit(
664: function permit(
```
***
## [NC-07] Use scientific notation (E.G. 1e18) rather than exponentiation (E.G. 10**18)
```
src/vault/adapter/yearn/YearnAdapter.sol
25: uint256 constant DEGRADATION_COEFFICIENT = 10**18;
```
***

## [NC-08] Avoid using low call function ecrecover
The ecrecover function is susceptible to signature malleability which could lead to replay attacks.
Use OZ library [ECDSA](https://docs.openzeppelin.com/contracts/4.x/api/utils#ECDSA) that its battle tested to avoid classic errors.

```
src/utils/MultiRewardStaking.sol
459: address recoveredAddress = ecrecover(
678: address recoveredAddress = ecrecover(
```
***

## [NC‑09] NatSpec is incomplete
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
In the following places there is unfinished NatSpec documentation:
```solidity
src/vault/adapter/yearn/YearnAdapter.sol
80: function _totalAssets() internal view override returns (uint256) {
89: function _shareValue(uint256 yShares) internal view returns (uint256) {
101: function _freeFunds() internal view returns (uint256) {
114: function _calculateLockedProfit() internal view returns (uint256) {
129: function convertToUnderlyingShares(uint256, uint256 shares)
144: function maxDeposit(address) public view override returns (uint256) {
166: function _protocolWithdraw(uint256 assets, uint256 shares)

src/vault/adapter/beefy/BeefyAdapter.sol
122: function convertToUnderlyingShares(uint256, uint256 shares)
148: function previewWithdraw(uint256 assets)
167: function previewRedeem(uint256 shares)

src/utils/MultiRewardStaking.sol
107: function _deposit(
121: function _withdraw(
137: function _transfer(

```
***

## [NC-10] You can use named parameters in mapping types
From Solidity [0.8.18](https://blog.soliditylang.org/2023/02/01/solidity-0.8.18-release-announcement/) you can use named parameters in mapping types. This will make the code much more readable.
For example check code below:
In `MultiRewardStaking.sol`

From:
```
FROM:
215:  // user => rewardToken -> rewardsIndex
216:  mapping(address => mapping(IERC20 => uint256)) public userIndex;
217:  // user => rewardToken -> accruedRewards
218:  mapping(address => mapping(IERC20 => uint256)) public accruedRewards;
```

To:
```
216:  mapping(address user => mapping(IERC20 rewardToken => uint256 rewardsIndex)) public userIndex;
218:  mapping(address user => mapping(IERC20 rewardToken => uint256 accruedRewards)) public accruedRewards;
```

This way you can put named parameters in mapping types everywhere in the code.