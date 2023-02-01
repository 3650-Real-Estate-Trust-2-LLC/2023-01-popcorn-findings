# QA REPORT

---

### Summary of low risk issues


| Number     | Issue details                                     | Instances |
| ------------ | --------------------------------------------------- | :---------: |
| [L-1](#L1) | Use of`block.timestamp`                           |    30    |
| [L-2](#L5) | Use`call()` instead of `transfer()`               |     1     |
| [L-3](#L6) | Lock pragmas to specific compiler version.        |    35    |
| [L-4](#L8) | Empty blocks should be removed or emit something. |     3     |

*Note: The instances of issue with id L-4 are not listed in the c4udit output.*

*Total: 4 issues.*

### Summary of non-critical issues


| Number        | Issue details                                                                         | Instances |
| --------------- | --------------------------------------------------------------------------------------- | :---------: |
| [NC-1](#NC3)  | Missing timelock for critical changes.                                                |    31    |
| [NC-2](#NC5)  | Use scientific notation (e.g.`1e18`) rather than exponentiation (e.g.`10**18`).       |     1     |
| [NC-3](#NC6)  | Lines are too long.                                                                   |     3     |
| [NC-4](#NC7)  | Add parameter to emitted event.                                                       |     1     |
| [NC-5](#NC9)  | Include`return` parameters in NatSpec comments.                                       |     3     |
| [NC-6](#NC10) | Use of`bytes.concat()` instead of `abi.encodePacked()`.                               |     7     |

*Total: 6 issues.*

---

### Low Risk Issues

### <a id=L1>[L-1]</a> Use of `block.timestamp`

##### Description

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the *[Entropy Illusion](https://hacken.io/discover/most-common-smart-contract-vulnerabilities/#Entropy_Illusion)* for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.

##### Recommendation

Use oracle instead of `block.timestamp`

##### *Instances (30):*

File: [2023-01-popcorn/src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L114 )

```solidity
114: uint32 start = block.timestamp.safeCastTo32() + offset;
163: escrows[escrowId].lastUpdateTime = block.timestamp.safeCastTo32();
175: block.timestamp.safeCastTo32() < escrow.start
181: (escrow.balance * (block.timestamp - uint256(escrow.lastUpdateTime))) /
```

File: [2023-01-popcorn/src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L276 )

```solidity
276: ? block.timestamp.safeCastTo32()
284: lastUpdatedTimestamp: block.timestamp.safeCastTo32()
309: prevEndTime > block.timestamp ? prevEndTime : block.timestamp.safeCastTo32(),
346: rewardInfos[rewardToken].lastUpdatedTimestamp = block.timestamp.safeCastTo32();
356: if (rewardsEndTimestamp > block.timestamp)
357: amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);
359: return (block.timestamp + (amount / uint256(rewardsPerSecond))).safeCastTo32();
392: if (rewards.rewardsEndTimestamp > block.timestamp) {
393: elapsed = block.timestamp - rewards.lastUpdatedTimestamp;
409: rewardInfos[_rewardToken].lastUpdatedTimestamp = block.timestamp.safeCastTo32();
454: if (deadline < block.timestamp) revert PermitDeadlineExpired(deadline);
```

File: [2023-01-popcorn/src/vault/Vault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L87 )

```solidity
87: feesUpdatedAt = block.timestamp;
94: abi.encodePacked("Popcorn", name(), block.timestamp, "Vault")
434: totalAssets() * (block.timestamp - feesUpdatedAt),
488: if (managementFee > 0) feesUpdatedAt = block.timestamp;
534: proposedFeeTime = block.timestamp;
536: emit NewFeesProposed(newFees, block.timestamp);
541: if (block.timestamp < proposedFeeTime + quitPeriod)
583: proposedAdapterTime = block.timestamp;
585: emit NewAdapterProposed(newAdapter, block.timestamp);
595: if (block.timestamp < proposedAdapterTime + quitPeriod)
673: if (deadline < block.timestamp) revert PermitDeadlineExpired(deadline);
```

File: [2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L86 )

```solidity
86: lastHarvest = block.timestamp;
441: ((lastHarvest + harvestCooldown) < block.timestamp)
641: if (deadline < block.timestamp) revert PermitDeadlineExpired(deadline);
```

File: [2023-01-popcorn/src/vault/adapter/yearn/YearnAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L115 )

```solidity
115: uint256 lockedFundsRatio = (block.timestamp - yVault.lastReport()) *
```

### <a id=L5>[L-2]</a> Use `call()` instead of `transfer()`

##### Description

The `transfer()` function only allows the recipient to use 2300 gas. If the recipient uses more than that, transfers will fail. In the future gas costs might change increasing the likelihood of that happening.
Keep in mind that `call()` introduces the risk of reentrancy, but as the code follows the CEI pattern it should be fine.

##### Recommendation

Replace `transfer()` calls with `call()`, keep in mind to check whether the call was successful by validating the return value.

##### *Instances (1):*

File: [2023-01-popcorn/src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L182 )

```solidity
182: _rewardTokens[i].transfer(user, rewardAmount);
```

### <a id=L6>[L-3]</a> Lock pragmas to specific compiler version.

##### Description

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.

##### Recommendation

Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version. [solidity-specific/locking-pragmas](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)

##### *Instances (35):*

File: [2023-01-popcorn/src/interfaces/IEIP165.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IEIP165.sol#L2 )

```solidity
2: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/interfaces/IMultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol#L3 )

```solidity
3: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/interfaces/IMultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L3 )

```solidity
3: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/interfaces/vault/IAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IAdapter.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/interfaces/vault/IAdminProxy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IAdminProxy.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/interfaces/vault/ICloneFactory.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ICloneFactory.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/interfaces/vault/ICloneRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ICloneRegistry.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/interfaces/vault/IDeploymentController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IDeploymentController.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/interfaces/vault/IERC4626.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IERC4626.sol#L3 )

```solidity
3: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/interfaces/vault/IPermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IPermissionRegistry.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/interfaces/vault/IStrategy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IStrategy.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/interfaces/vault/ITemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ITemplateRegistry.sol#L3 )

```solidity
3: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/interfaces/vault/IVault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol#L3 )

```solidity
3: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/interfaces/vault/IVaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVaultController.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/interfaces/vault/IVaultRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVaultRegistry.sol#L3 )

```solidity
3: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/interfaces/vault/IWithRewards.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IWithRewards.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/utils/EIP165.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/EIP165.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/vault/AdminProxy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/AdminProxy.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/vault/CloneFactory.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneFactory.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/vault/CloneRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/vault/DeploymentController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/vault/PermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/vault/TemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/vault/Vault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/vault/VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L3 )

```solidity
3: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/vault/VaultRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/vault/adapter/abstracts/OnlyStrategy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/OnlyStrategy.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/vault/adapter/abstracts/WithRewards.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/WithRewards.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/vault/adapter/beefy/BeefyAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/vault/adapter/beefy/IBeefy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/IBeefy.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/vault/adapter/yearn/IYearn.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/IYearn.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

File: [2023-01-popcorn/src/vault/adapter/yearn/YearnAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L4 )

```solidity
4: pragma solidity ^0.8.15;
```

### <a id=L8>[L-4]</a> Empty blocks should be removed or emit something.

##### Description

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified
```solidity if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}} ```

##### Recommendation

Empty blocks should be removed or emit something (The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

##### *Instances (3):*

File: [2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L258 )

```solidity
258: function _totalAssets() internal view virtual returns (uint256) {}
265: function _underlyingBalance() internal view virtual returns (uint256) {}
276: {}
```

### Non-critical Issues

### <a id=NC3>[NC-1]</a> Missing timelock for critical changes.

##### Description

A timelock should be added to functions that perform critical changes.

##### Recommendation

TODO: Add Timelock for the following functions.

##### *Instances (31):*

File: [2023-01-popcorn/src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L207 )

```solidity
207: function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {
```

File: [2023-01-popcorn/src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L251 )

```solidity
251: ) external onlyOwner {
296: function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {
```

File: [2023-01-popcorn/src/vault/CloneRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol#L45 )

```solidity
45: ) external onlyOwner {
```

File: [2023-01-popcorn/src/vault/DeploymentController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol#L55 )

```solidity
55: function addTemplateCategory(bytes32 templateCategory) external onlyOwner {
80: function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {
121: function nominateNewDependencyOwner(address _owner) external onlyOwner {
```

File: [2023-01-popcorn/src/vault/PermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol#L38 )

```solidity
38: function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {
```

File: [2023-01-popcorn/src/vault/TemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol#L52 )

```solidity
52: function addTemplateCategory(bytes32 templateCategory) external onlyOwner {
71: ) external onlyOwner {
102: function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {
```

File: [2023-01-popcorn/src/vault/Vault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L525 )

```solidity
525: function proposeFees(VaultFees calldata newFees) external onlyOwner {
553: function setFeeRecipient(address _feeRecipient) external onlyOwner {
578: function proposeAdapter(IERC4626 newAdapter) external onlyOwner {
629: function setQuitPeriod(uint256 _quitPeriod) external onlyOwner {
643: function pause() external onlyOwner {
648: function unpause() external onlyOwner {
```

File: [2023-01-popcorn/src/vault/VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L408 )

```solidity
408: function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {
543: function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {
561: function addTemplateCategories(bytes32[] calldata templateCategories) external onlyOwner {
723: function nominateNewAdminProxyOwner(address newOwner) external onlyOwner {
751: function setPerformanceFee(uint256 newFee) external onlyOwner {
764: function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {
791: function setHarvestCooldown(uint256 newCooldown) external onlyOwner {
804: function setAdapterHarvestCooldowns(address[] calldata adapters) external onlyOwner {
832: function setDeploymentController(IDeploymentController _deploymentController) external onlyOwner {
864: function setActiveTemplateId(bytes32 templateCategory, bytes32 templateId) external onlyOwner {
```

File: [2023-01-popcorn/src/vault/VaultRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol#L44 )

```solidity
44: function registerVault(VaultMetadata calldata _metadata) external onlyOwner {
```

File: [2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L500 )

```solidity
500: function setHarvestCooldown(uint256 newCooldown) external onlyOwner {
574: function pause() external onlyOwner {
582: function unpause() external onlyOwner {
```

### <a id=NC5>[NC-2]</a> Use scientific notation (e.g.`1e18`) rather than exponentiation (e.g.`10**18`).

##### Description

Use scientific notation (e.g.`1e18`) rather than exponentiation (e.g.`10**18`) that could lead to errors.

##### Recommendation

Use scientific notation instead of exponentiation.

##### *Instances (1):*

File: [2023-01-popcorn/src/vault/adapter/yearn/YearnAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L25 )

```solidity
25: uint256 constant DEGRADATION_COEFFICIENT = 10**18;
```

### <a id=NC6>[NC-3]</a> Lines are too long.

##### Description

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length

##### Recommendation

Reduce number of characters per line to improve readability.

##### *Instances (3):*

File: [2023-01-popcorn/src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L49 )

```solidity
49: * @dev there is no check to ensure that all escrows are owned by the same account. Make sure to account for this either by only sending ids for a specific account or by filtering the Escrows by account later on.
```

File: [2023-01-popcorn/src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L7 )

```solidity
7: import { ERC4626Upgradeable, ERC20Upgradeable, IERC20Upgradeable as IERC20, IERC20MetadataUpgradeable as IERC20Metadata } from "openzeppelin-contracts-upgradeable/token/ERC20/extensions/ERC4626Upgradeable.sol";
```

File: [2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L6 )

```solidity
6: import {ERC4626Upgradeable, IERC20Upgradeable as IERC20, IERC20MetadataUpgradeable as IERC20Metadata, ERC20Upgradeable as ERC20} from "openzeppelin-contracts-upgradeable/token/ERC20/extensions/ERC4626Upgradeable.sol";
```

### <a id=NC7>[NC-4]</a> Add parameter to emitted event.

##### Description

No parameter has been passed to an emitted event. Add some parameter to better depict what has been done on the blockchain.

##### Recommendation

Consider passing a parameter to the event.

##### *Instances (1):*

File: [2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L449 )

```solidity
449: emit Harvested();
```

### <a id=NC9>[NC-5]</a> Include `return` parameters in NatSpec comments.

##### Description

If Return parameters are declared, you must prefix them with `/// @return`.Some code analysis programs do analysis by reading NatSpec details, if they can't see the `@return` tag, they do incomplete analysis.
Recommendation Code Style:

```Solidity
/// @notice information about what a function does
/// @param pageId The id of the page to get the URI for.
/// @return Returns a page's URI if it has been minted
function tokenURI(uint256 pageId) public view virtual override returns (string memory) {
if (pageId == 0 || pageId > currentId) revert("NOT_MINTED");
return string.concat(BASE_URI, pageId.toString());
}
```

##### Recommendation

Include `return` parameters in NatSpec comments.

##### *Instances (3):*

File: [2023-01-popcorn/src/vault/AdminProxy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/AdminProxy.sol#L18 )

```solidity
18: /// @notice Execute arbitrary management functions.
```

File: [2023-01-popcorn/src/vault/Vault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L472 )

```solidity
472: /// @notice Minimal function to call `takeFees` modifier.
```

File: [2023-01-popcorn/src/vault/VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L241 )

```solidity
241: /// @notice Encodes adapter init call. Was moved into its own function to fix "stack too deep" error.
```

### <a id=NC10>[NC-6]</a> Use of `bytes.concat()` instead of `abi.encodePacked()`.

##### Description

Rather than using `abi.encodePacked` for appending bytes, since version `0.8.4`, `bytes.concat()` is enabled.

##### Recommendation

Since version `0.8.4` for appending bytes, `bytes.concat()` can be used instead of `abi.encodePacked()`.

##### *Instances (7):*

File: [2023-01-popcorn/src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L104 )

```solidity
104: bytes32 id = keccak256(abi.encodePacked(token, account, amount, nonce));
```

File: [2023-01-popcorn/src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L49 )

```solidity
49: _name = string(abi.encodePacked("Staked ", IERC20Metadata(address(_stakingToken)).name()));
50: _symbol = string(abi.encodePacked("pst-", IERC20Metadata(address(_stakingToken)).symbol()));
461: abi.encodePacked(
```

File: [2023-01-popcorn/src/vault/Vault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L94 )

```solidity
94: abi.encodePacked("Popcorn", name(), block.timestamp, "Vault")
680: abi.encodePacked(
```

File: [2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L648 )

```solidity
648: abi.encodePacked(
```
