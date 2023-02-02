# Gas Optimizations Report

|         | Issue                                                                                                                              | Instances |
| ------- | :--------------------------------------------------------------------------------------------------------------------------------- | :-------: |
| [G-001] | x += y or x -= y costs more gas than x = x + y or x = x - y for state variables                                                    |     5     |
| [G-002] | `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping` |     3     |
| [G-003] | Functions guaranteed to revert when called by normal users can be marked payable                                                   |    35     |
| [G-004] | Don't use `_msgSender()` if not supporting EIP-2771                                                                                |     4     |
| [G-005] | Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate                 |     5     |
| [G-006] | Using `calldata` instead of memory for read-only arguments in external functions saves gas                                         |     4     |
| [G-007] | Multiple accesses of a `mapping/array` should use a local variable cache                                                           |    14     |
| [G-008] | internal functions only called once can be inlined to save gas                                                                     |     9     |
| [G-009] | Replace modifier with function                                                                                                     |     5     |
| [G-010] | Use named returns for local variables where it is possible.                                                                        |     5     |

## [G-001] x += y or x -= y costs more gas than x = x + y or x = x - y for state variables

### Impact

Using the addition operator instead of plus-equals saves 113 gas. Usually does not work with struct and mappings.

### Findings

Total:5

[src/utils/MultiRewardEscrow.sol#L110](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L110)

```solidity
110:    amount -= fee;
```

[src/utils/MultiRewardStaking.sol#L357](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L357)

```solidity
357:    amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);
```

[src/vault/Vault.sol#L228](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L228)

```solidity
228:    shares += feeShares;
```

[src/vault/adapter/abstracts/AdapterBase.sol#L225](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L225)

```solidity
225:    underlyingBalance -= underlyingBalance_ - _underlyingBalance();
```

[src/vault/adapter/abstracts/AdapterBase.sol#L158](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L158)

```solidity
158:    underlyingBalance += _underlyingBalance() - underlyingBalance_;
```

## [G-002] `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping`

### Impact

It may not be obvious, but every time you copy a storage `struct/array/mapping` to a `memory` variable, you are literally copying each member by reading it from `storage`, which is expensive. And when you use the `storage` keyword, you are just storing a pointer to the storage, which is much cheaper.

### Findings

Total:3

[src/vault/adapter/beefy/BeefyAdapter.sol#L137](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L137)

```solidity
137:    address[] memory _rewardTokens = new address[](1);
```

[src/vault/VaultController.sol#L155](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L155)

```solidity
155:    address[] memory vaultContracts = new address[](1);
```

[src/vault/VaultController.sol#L156](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L156)

```solidity
156:    bytes[] memory rewardsDatas = new bytes[](1);
```

### Recommendation

Using `storage` instead of `memory`

## [G-003] Functions guaranteed to revert when called by normal users can be marked payable

### Impact

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
The extra opcodes avoided are `CALLVALUE(2)`,`DUP1(3)`,`ISZERO(3)`,`PUSH2(3)`,`JUMPI(10)`,`PUSH1(3)`,`DUP1(3)`,`REVERT(0)`,`JUMPDEST(1)`,`POP(2)`, which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

### Findings

Total:35

[src/vault/adapter/abstracts/WithRewards.sol#L15](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/WithRewards.sol#L15)

```solidity
15:    function claim() public virtual onlyStrategy {}
```

[src/vault/CloneFactory.sol#L39](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneFactory.sol#L39)

```solidity
39:    function deploy(Template calldata template, bytes calldata data) external onlyOwner returns (address clone) {
```

[src/vault/PermissionRegistry.sol#L38](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L38)

```solidity
38:    function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {
```

[src/vault/CloneRegistry.sol#L45](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneRegistry.sol#L45)

```solidity
45:    ) external onlyOwner {
```

[src/vault/VaultRegistry.sol#L44](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L44)

```solidity
44:    function registerVault(VaultMetadata calldata _metadata) external onlyOwner {
```

[src/vault/DeploymentController.sol#L80](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L80)

```solidity
80:    function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {
```

[src/vault/DeploymentController.sol#L121](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L121)

```solidity
121:    function nominateNewDependencyOwner(address _owner) external onlyOwner {
```

[src/vault/DeploymentController.sol#L55](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L55)

```solidity
55:    function addTemplateCategory(bytes32 templateCategory) external onlyOwner {
```

[src/vault/DeploymentController.sol#L103](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L103)

```solidity
103:    ) external onlyOwner returns (address clone) {
```

[src/vault/TemplateRegistry.sol#L102](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L102)

```solidity
102:    function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {
```

[src/vault/TemplateRegistry.sol#L52](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L52)

```solidity
52:    function addTemplateCategory(bytes32 templateCategory) external onlyOwner {
```

[src/vault/TemplateRegistry.sol#L71](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L71)

```solidity
71:    ) external onlyOwner {
```

[src/utils/MultiRewardEscrow.sol#L207](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L207)

```solidity
207:    function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {
```

[src/utils/MultiRewardStaking.sol#L296](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L296)

```solidity
296:    function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {
```

[src/utils/MultiRewardStaking.sol#L251](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L251)

```solidity
251:    ) external onlyOwner {
```

[src/vault/Vault.sol#L525](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L525)

```solidity
525:    function proposeFees(VaultFees calldata newFees) external onlyOwner {
```

[src/vault/Vault.sol#L629](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L629)

```solidity
629:    function setQuitPeriod(uint256 _quitPeriod) external onlyOwner {
```

[src/vault/Vault.sol#L578](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L578)

```solidity
578:    function proposeAdapter(IERC4626 newAdapter) external onlyOwner {
```

[src/vault/Vault.sol#L643](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L643)

```solidity
643:    function pause() external onlyOwner {
```

[src/vault/Vault.sol#L553](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L553)

```solidity
553:    function setFeeRecipient(address _feeRecipient) external onlyOwner {
```

[src/vault/Vault.sol#L648](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L648)

```solidity
648:    function unpause() external onlyOwner {
```

[src/vault/VaultController.sol#L723](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L723)

```solidity
723:    function nominateNewAdminProxyOwner(address newOwner) external onlyOwner {
```

[src/vault/VaultController.sol#L791](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L791)

```solidity
791:    function setHarvestCooldown(uint256 newCooldown) external onlyOwner {
```

[src/vault/VaultController.sol#L764](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L764)

```solidity
764:    function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {
```

[src/vault/VaultController.sol#L543](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L543)

```solidity
543:    function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {
```

[src/vault/VaultController.sol#L408](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L408)

```solidity
408:    function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {
```

[src/vault/VaultController.sol#L832](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L832)

```solidity
832:    function setDeploymentController(IDeploymentController _deploymentController) external onlyOwner {
```

[src/vault/VaultController.sol#L864](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L864)

```solidity
864:    function setActiveTemplateId(bytes32 templateCategory, bytes32 templateId) external onlyOwner {
```

[src/vault/VaultController.sol#L561](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L561)

```solidity
561:    function addTemplateCategories(bytes32[] calldata templateCategories) external onlyOwner {
```

[src/vault/VaultController.sol#L751](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L751)

```solidity
751:    function setPerformanceFee(uint256 newFee) external onlyOwner {
```

[src/vault/VaultController.sol#L804](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L804)

```solidity
804:    function setAdapterHarvestCooldowns(address[] calldata adapters) external onlyOwner {
```

[src/vault/adapter/abstracts/AdapterBase.sol#L582](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L582)

```solidity
582:    function unpause() external onlyOwner {
```

[src/vault/adapter/abstracts/AdapterBase.sol#L549](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L549)

```solidity
549:    function setPerformanceFee(uint256 newFee) public onlyOwner {
```

[src/vault/adapter/abstracts/AdapterBase.sol#L500](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L500)

```solidity
500:    function setHarvestCooldown(uint256 newCooldown) external onlyOwner {
```

[src/vault/adapter/abstracts/AdapterBase.sol#L574](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L574)

```solidity
574:    function pause() external onlyOwner {
```

## [G-004] Don't use `_msgSender()` if not supporting EIP-2771

### Impact

The use of `_msgSender()` when there is no implementation of a meta transaction mechanism that uses it, such as EIP-2771, very slightly increases gas consumption.

### Findings

Total:4

[src/vault/adapter/abstracts/AdapterBase.sol#L119](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L119)

```solidity
119:    _deposit(_msgSender(), receiver, assets, shares);
```

[src/vault/adapter/abstracts/AdapterBase.sol#L138](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L138)

```solidity
138:    _deposit(_msgSender(), receiver, assets, shares);
```

[src/vault/adapter/abstracts/AdapterBase.sol#L182](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L182)

```solidity
182:    _withdraw(_msgSender(), receiver, owner, assets, shares);
```

[src/vault/adapter/abstracts/AdapterBase.sol#L201](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L201)

```solidity
201:    _withdraw(_msgSender(), receiver, owner, assets, shares);
```

### Recommendation

Replace `_msgSender()` with `msg.sender` if there is no mechanism to support meta-transactions like EIP-2771 implemented.

## [G-005] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

### Impact

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

### Findings

Total:5

[src/vault/CloneRegistry.sol#L28-L30](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneRegistry.sol#L28-L30)

```solidity
28:    mapping(address => bool) public cloneExists;
29:      // TemplateCategory => TemplateId => Clones
30:      mapping(bytes32 => mapping(bytes32 => address[])) public clones;
```

[src/vault/VaultRegistry.sol#L28-L31](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L28-L31)

```solidity
28:    mapping(address => VaultMetadata) public metadata;
29:
30:      // asset to vault addresses
31:      mapping(address => address[]) public vaultsByAsset;
```

[src/utils/MultiRewardEscrow.sol#L64-L67](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L64-L67)

```solidity
64:    mapping(bytes32 => Escrow) public escrows;
65:
66:      // User => Escrows
67:      mapping(address => bytes32[]) public userEscrowIds;
```

[src/utils/MultiRewardStaking.sol#L216-L218](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L216-L218)

```solidity
216:    mapping(address => mapping(IERC20 => uint256)) public userIndex;
217:      // user => rewardToken -> accruedRewards
218:      mapping(address => mapping(IERC20 => uint256)) public accruedRewards;
```

[src/utils/MultiRewardStaking.sol#L211-L213](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L211-L213)

```solidity
211:    mapping(IERC20 => RewardInfo) public rewardInfos;
212:      // rewardToken -> EscrowInfo
213:      mapping(IERC20 => EscrowInfo) public escrowInfos;
```

## [G-006] Using `calldata` instead of memory for read-only arguments in external functions saves gas

### Impact

When a function with a `memory` array is called externally, the `abi.decode()` step has to use a `for-loop` to copy each index of the `calldata` to the `memory` index. Each iteration of this `for-loop` costs at least 60 gas (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having `memory` arguments, it's still valid for implementation contracs to use `calldata` arguments instead.<br><br> If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it's still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one.

### Findings

Total:4

[src/interfaces/vault/IVaultController.sol#L57-L60](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVaultController.sol#L57-L60)

```solidity
57:    function changeStakingRewardsSpeeds(
58:        address[] memory vaults,
59:        IERC20[] memory rewardTokens,
60:        uint160[] memory rewardsSpeeds
```

[src/interfaces/vault/IVaultController.sol#L63-L66](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVaultController.sol#L63-L66)

```solidity
63:    function changeStakingRewardsSpeeds(
64:        address[] memory vaults,
65:        IERC20[] memory rewardTokens,
66:        uint160[] memory rewardsSpeeds
```

[src/interfaces/vault/IVaultController.sol#L57-L60](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVaultController.sol#L57-L60)

```solidity
57:    function fundStakingRewards(
58:        address[] memory vaults,
59:        IERC20[] memory rewardTokens,
60:        uint256[] memory amounts
```

[src/interfaces/vault/IVaultController.sol#L63-L66](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVaultController.sol#L63-L66)

```solidity
63:    function fundStakingRewards(
64:        address[] memory vaults,
65:        IERC20[] memory rewardTokens,
66:        uint256[] memory amounts
```

## [G-007] Multiple accesses of a `mapping/array` should use a local variable cache

### Impact

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into `memory/calldata` 

### Findings

Total:14

[src/vault/PermissionRegistry.sol#L47](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L47)

```solidity
47:    permissions[targets[i]] = newPermissions[i];
```

[src/vault/CloneRegistry.sol#L46](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneRegistry.sol#L46)

```solidity
46:    cloneExists[clone] = true;
```

[src/vault/VaultRegistry.sol#L47](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L47)

```solidity
47:    metadata[_metadata.vault] = _metadata;
```

[src/vault/TemplateRegistry.sol#L55](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L55)

```solidity
55:    templateCategoryExists[templateCategory] = true;
```

[src/vault/TemplateRegistry.sol#L76](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L76)

```solidity
76:    templates[templateCategory][templateId] = template;
```

[src/vault/TemplateRegistry.sol#L79](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L79)

```solidity
79:    templateExists[templateId] = true;
```

[src/utils/MultiRewardEscrow.sol#L116](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L116)

```solidity
116:    escrows[id] = Escrow({
```

[src/utils/MultiRewardStaking.sol#L266](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L266)

```solidity
266:    escrowInfos[rewardToken] = EscrowInfo({
```

[src/utils/MultiRewardStaking.sol#L279](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L279)

```solidity
279:    rewardInfos[rewardToken] = RewardInfo({
```

[src/utils/MultiRewardStaking.sol#L429](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L429)

```solidity
429:    userIndex[_user][_rewardToken] = rewards.index;
```

[src/utils/MultiRewardStaking.sol#L186](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L186)

```solidity
186:    accruedRewards[user][_rewardTokens[i]] = 0;
```

[src/vault/VaultController.sol#L68](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L68)

```solidity
68:    activeTemplateId[STAKING] = "MultiRewardStaking";
```

[src/vault/VaultController.sol#L69](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L69)

```solidity
69:    activeTemplateId[VAULT] = "V1";
```

[src/vault/VaultController.sol#L870](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L870)

```solidity
870:    activeTemplateId[templateCategory] = templateId;
```

## [G-008] internal functions only called once can be inlined to save gas

### Impact

Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.

### Findings

Total:9

[src/vault/adapter/yearn/YearnAdapter.sol#L89](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L89)

```solidity
89:    function _shareValue(uint256 yShares) internal view returns (uint256) {
```

[src/vault/adapter/yearn/YearnAdapter.sol#L109](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L109)

```solidity
109:    function _yTotalAssets() internal view returns (uint256) {
```

[src/vault/adapter/yearn/YearnAdapter.sol#L101](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L101)

```solidity
101:    function _freeFunds() internal view returns (uint256) {
```

[src/vault/adapter/yearn/YearnAdapter.sol#L114](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L114)

```solidity
114:    function _calculateLockedProfit() internal view returns (uint256) {
```

[src/vault/VaultController.sol#L692](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L692)

```solidity
692:    function _verifyAdapterConfiguration(address adapter, bytes32 adapterId) internal view {
```

[src/vault/VaultController.sol#L390](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L390)

```solidity
390:    function _registerVault(address vault, VaultMetadata memory metadata) internal {
```

[src/vault/VaultController.sol#L154](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L154)

```solidity
154:    function _handleVaultStakingRewards(address vault, bytes memory rewardsData) internal {
```

[src/vault/adapter/abstracts/AdapterBase.sol#L479](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L479)

```solidity
479:    function _verifyAndSetupStrategy(bytes4[8] memory requiredSigs) internal {
```

[src/vault/adapter/abstracts/AdapterBase.sol#L258](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L258)

```solidity
258:    function _totalAssets() internal view virtual returns (uint256) {}
```

## [G-009] Replace modifier with function

### Impact

Modifiers make code more elegant, but cost more than normal functions.

### Findings

Total:5

[src/vault/adapter/abstracts/OnlyStrategy.sol#L10](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/OnlyStrategy.sol#L10)

```solidity
10:    modifier onlyStrategy() {
```

[src/vault/Vault.sol#L496](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L496)

```solidity
496:    modifier syncFeeCheckpoint() {
```

[src/vault/Vault.sol#L480](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L480)

```solidity
480:    modifier takeFees() {
```

[src/vault/VaultController.sol#L704](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L704)

```solidity
704:    modifier canCreate() {
```

[src/vault/adapter/abstracts/AdapterBase.sol#L559](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L559)

```solidity
559:    modifier takeFees() {
```

## [G-010] Use named returns for local variables where it is possible.

### Impact

It is not necessary to have both a named return and a return statement.

### Findings

Total:5

[src/vault/adapter/beefy/BeefyAdapter.sol#L173](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L173)

```solidity
173:    uint256 assets = _convertToAssets(shares, Math.Rounding.Down);
```

[src/vault/adapter/abstracts/AdapterBase.sol#L118](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L118)

```solidity
118:    uint256 shares = _previewDeposit(assets);
```

[src/vault/adapter/abstracts/AdapterBase.sol#L180](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L180)

```solidity
180:    uint256 shares = _previewWithdraw(assets);
```

[src/vault/adapter/abstracts/AdapterBase.sol#L137](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L137)

```solidity
137:    uint256 assets = _previewMint(shares);
```

[src/vault/adapter/abstracts/AdapterBase.sol#L200](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L200)

```solidity
200:    uint256 assets = _previewRedeem(shares);
```
