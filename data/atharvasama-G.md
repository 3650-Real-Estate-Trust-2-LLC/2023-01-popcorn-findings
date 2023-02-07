# Gas Optimizations list

| Number | Details                                                                           | Instances |
| ------ | --------------------------------------------------------------------------------- | --------- |
| 1      | ```x += y``` costs more gas than ```x = x + y``` for state variables              | 2         |
| 2      | Can declare the variable outside the loop to save gas                             | 34         |
| 3      | Multiple address/id mappings can be combined into a single mapping to a struct    | 7        |
| 4      | Internal functions only called once can be inlined to save gas                    | 8         |
| 5      | ```keccak256()``` should only need to be called on a specific string literal once | 9        |






# Gas Optimizations
## [G-01] ```x += y``` costs more gas than ```x = x + y``` for state variables.
Using the addition operator instead of plus-equals saves some gas for state variables.
>[src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol)
>```js
>408:     rewardInfos[_rewardToken].index += deltaIndex;
>
>431:     accruedRewards[_user][_rewardToken] += supplierDelta;
>```

## [G-02] Can make variable outside loop to save gas
>[src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol)
>```js
>156:       bytes32 escrowId = escrowIds[i];
>157:       Escrow memory escrow = escrows[escrowId];
>159:       uint256 claimable = _getClaimableAmount(escrow);
>```
>
>[src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol)
>```js
>172:       uint256 rewardAmount = accruedRewards[user][_rewardTokens[i]];
>176:       EscrowInfo memory escrowInfo = escrowInfos[_rewardTokens[i]];
>375:       RewardInfo memory rewards = rewardInfos[rewardToken];
>374:       IERC20 rewardToken = _rewardTokens[i];
>```
>
>[src/vault/VaultController.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol)
>```js
>438:       (
>439:         address rewardsToken,
>440:         uint160 rewardsPerSecond,
>441:         uint256 amount,
>442:         bool useEscrow,
>443:         uint224 escrowDuration,
>444:         uint24 escrowPercentage,
>445:         uint256 offset
>446:       ) = abi.decode(rewardTokenData[i], (address, uint160, uint256, bool, uint224, uint24, uint256));
>447:       _verifyToken(rewardsToken);
>
>450:       (bool success, bytes memory returnData) = adminProxy.execute(
>451:         rewardsToken,
>452:         abi.encodeWithSelector(IERC20.approve.selector, staking, type(uint256).max)
>453:       );
>
>497:       (bool success, bytes memory returnData) = adminProxy.execute(
>498:         staking,
>499:         abi.encodeWithSelector(IMultiRewardStaking.changeRewardSpeed.selector, rewardTokens[i], rewardsSpeeds[i])
>500:       );
>
>565:       (bool success, bytes memory returnData) = adminProxy.execute(
>566:         _deploymentController,
>567:         abi.encodeWithSelector(IDeploymentController.addTemplateCategory.selector, templateCategories[i])
>568:       );
>
>588:       (bool success, bytes memory returnData) = adminProxy.execute(
>589:         address(_deploymentController),
>590:         abi.encodeWithSelector(
>591:           ITemplateRegistry.toggleTemplateEndorsement.selector,
>592:           templateCategories[i],
>593:           templateIds[i]
>594:         )
>595:       );
>
>609:       (bool success, bytes memory returnData) = adminProxy.execute(
>610:         IVault(vaults[i]).adapter(),
>611:         abi.encodeWithSelector(IPausable.pause.selector)
>612:       );
>
>622:       (bool success, bytes memory returnData) = adminProxy.execute(
>623:         vaults[i],
>624:         abi.encodeWithSelector(IPausable.pause.selector)
>625:       );
>
>635:       (bool success, bytes memory returnData) = adminProxy.execute(
>636:         IVault(vaults[i]).adapter(),
>637:         abi.encodeWithSelector(IPausable.unpause.selector)
>638:       );
>
>648:       (bool success, bytes memory returnData) = adminProxy.execute(
>649:         vaults[i],
>650:         abi.encodeWithSelector(IPausable.unpause.selector)
>651:       );
>
>767:       (bool success, bytes memory returnData) = adminProxy.execute(
>768:         adapters[i],
>769:         abi.encodeWithSelector(IAdapter.setPerformanceFee.selector, performanceFee)
>770:       );
>
>807:       (bool success, bytes memory returnData) = adminProxy.execute(
>808:         adapters[i],
>809:         abi.encodeWithSelector(IAdapter.setHarvestCooldown.selector, harvestCooldown)
>810:       );
>```

## [G-03] Multiple address/id mappings can be combined into a single mapping to a struct
If both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Depending on the circumstances and sizes of types, can avoid a Gsset per mapping combined.

>[src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol)
>```js
>66:   // User => Escrows
>67:   mapping(address => bytes32[]) public userEscrowIds;
>68:   // User => RewardsToken => Escrows
>69:   mapping(address => mapping(IERC20 => bytes32[])) public userEscrowIdsByToken;
>```
>
>[src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol)
>```js
>215:   // user => rewardToken -> rewardsIndex
>216:   mapping(address => mapping(IERC20 => uint256)) public userIndex;
>217:   // user => rewardToken -> accruedRewards
>218:   mapping(address => mapping(IERC20 => uint256)) public accruedRewards;
>```
>
>[src/vault/TemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol)
>```js
>30:   // TemplateCategory => TemplateId => Template
>31:   mapping(bytes32 => mapping(bytes32 => Template)) public templates;
>32:   mapping(bytes32 => bytes32[]) public templateIds;
>33:   mapping(bytes32 => bool) public templateExists;
>```

## [G-04] Internal functions only called once can be inlined to save gas 
>[src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol)
>```js
>191:   function _lockToken(
>192:     address user,
>193:     IERC20 rewardToken,
>194:     uint256 rewardAmount,
>195:     EscrowInfo memory escrowInfo
>196:   ) internal {
>```
>
>[src/vault/VaultController.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol)
>```js
>120:   function _deployVault(VaultInitParams memory vaultData, IDeploymentController _deploymentController)
>121:     internal
>122:     returns (address vault)
>123:   {
>
>141:   function _registerCreatedVault(
>142:     address vault,
>143:     address staking,
>144:     VaultMetadata memory metadata
>145:   ) internal {
>146:     metadata.vault = vault;
>147:     metadata.staking = staking;
>148:     metadata.creator = msg.sender;
>149: 
>150:     _registerVault(vault, metadata);
>151:   }
>
>154:   function _handleVaultStakingRewards(address vault, bytes memory rewardsData) internal {
>
>242:   function _encodeAdapterData(DeploymentArgs memory adapterData, bytes memory baseAdapterData)
>243:     internal
>244:     returns (bytes memory)
>
>256:   function _deployStrategy(DeploymentArgs memory strategyData, IDeploymentController _deploymentController)
>257:     internal
>258:     returns (address strategy)
>259:   {
>
>390:   function _registerVault(address vault, VaultMetadata memory metadata) internal {
>
>692:   function _verifyAdapterConfiguration(address adapter, bytes32 adapterId) internal view {
>```

## [G-05] ```keccak256()``` should only need to be called on a specific string literal once
It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once

>[src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol)
>```js
>466:                 keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),
>
>495:           keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
>497:           keccak256("1"),
>```
>
>[src/vault/Vault.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)
>```js
>685:                                 keccak256(
>686:                                     "Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"
>687:                                 ),
>
>720:                     keccak256(
>721:                         "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"
>722:                     ),
>724:                     keccak256("1"),
>```
>
>[src/vault/adapter/abstracts/AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol)
>```js
>653:                                 keccak256(
>654:                                     "Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"
>655:                                 ),
>
>688:                     keccak256(
>689:                         "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"
>690:                     ),
>692:                     keccak256("1"),
>```
