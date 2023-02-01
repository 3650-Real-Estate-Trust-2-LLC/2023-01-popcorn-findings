# GAS OPTIMIZATION REPORT

---

### Summary of optimizations


| Number     | Issue details                                                                                                                                         | Instances |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------: |
| [G-1](#G1) | `++i/i++` should be `unchecked{++i}`/`unchecked{i++}` when  it is not possible for them to overflow, for example when used in `for` and `while` loops |    21    |
| [G-2](#G2) | Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate.                                         |     3     |
| [G-3](#G3) | Using storage instead of memory for structs/arrays saves gas.                                                                                         |     5     |
| [G-4](#G4) | Usage of`uint`s/`int`s smaller than 32 bytes (256 bits) incurs overhead.                                                                              |    62    |
| [G-5](#G5) | `abi.encode()` is less efficient than `abi.encodePacked()`                                                                                            |     7     |
| [G-6](#G6) | Internal functions only called once can be inlined to save gas.                                                                                       |    15    |
| [G-7](#G7) | Multiple accesses of a mapping/array should use a local variable cache.                                                                               |     4     |
| [G-8](#G8) | `>=` costs less gas than `>`.                                                                                                                         |    38    |
| [G-9](#G9) | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables                                                                                |    12    |

*Total: 9 issues.*

---

### Gas Optimizations

### <a id=G1>[G-1]</a> `++i/i++` should be `unchecked{++i}`/`unchecked{i++}` when  it is not possible for them to overflow, for example when used in `for` and `while` loops

##### Description

In Solidity 0.8+, there's a default overflow check on unsigned integers. It's possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck [cannot be made inline](https://github.com/ethereum/solidity/issues/10695).
Example for loop:

```Solidity
for (uint i = 0; i < length; i++) {
// do something that doesn't change the value of i
}
```

In this example, the for loop post condition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of i to enter the for-loop body `is 2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks. You should manually do this by:

```Solidity
for (uint i = 0; i < length; i = unchecked_inc(i)) {
	// do something that doesn't change the value of i
}

function unchecked_inc(uint i) returns (uint) {
	unchecked {
		return i + 1;
	}
}
```

Or just:

```Solidity
for (uint i = 0; i < length;) {
	// do something that doesn't change the value of i
	unchecked { i++; 	}
}
```

Note that it’s important that the call to `unchecked_inc` is inlined. This is only possible for solidity versions starting from `0.8.2`.

Gas savings: roughly speaking this can save 30-40 gas per loop iteration. For lengthy loops, this can be significant!
(This is only relevant if you are using the default solidity checked arithmetic.)

##### Recommendation

Use the `unchecked` keyword

##### *Instances (21):*

File: [2023-01-popcorn/src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L53 )

```solidity
53: for (uint256 i = 0; i < escrowIds.length; i++) {
155: for (uint256 i = 0; i < escrowIds.length; i++) {
210: for (uint256 i = 0; i < tokens.length; i++) {
```

File: [2023-01-popcorn/src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L171 )

```solidity
171: for (uint8 i; i < _rewardTokens.length; i++) {
373: for (uint8 i; i < _rewardTokens.length; i++) {
```

File: [2023-01-popcorn/src/vault/PermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol#L42 )

```solidity
42: for (uint256 i = 0; i < len; i++) {
```

File: [2023-01-popcorn/src/vault/VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L319 )

```solidity
319: for (uint8 i = 0; i < len; i++) {
337: for (uint8 i = 0; i < len; i++) {
357: for (uint8 i = 0; i < len; i++) {
374: for (uint8 i = 0; i < len; i++) {
437: for (uint256 i = 0; i < len; i++) {
494: for (uint256 i = 0; i < len; i++) {
523: for (uint256 i = 0; i < len; i++) {
564: for (uint256 i = 0; i < len; i++) {
587: for (uint256 i = 0; i < len; i++) {
607: for (uint256 i = 0; i < len; i++) {
620: for (uint256 i = 0; i < len; i++) {
633: for (uint256 i = 0; i < len; i++) {
646: for (uint256 i = 0; i < len; i++) {
766: for (uint256 i = 0; i < len; i++) {
806: for (uint256 i = 0; i < len; i++) {
```

### <a id=G2>[G-2]</a> Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate.

##### Description

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

For example:

```Solidity
mapping(address => uint256) public nonces;
-mapping (address => bool) public minters;
-mapping (address => bool) public markets;
+mapping (address => uint256) public markets_minters;
+            // address => uint256 => 1 (minters true)
+            // address => uint256 => 2 (minters false)
+            // address => uint256 => 3 (markets true)
+            // address => uint256 => 4 (markets false)
```

##### Recommendation

Where appropriate, you can combine multiple address mappings into a single mapping of an address to a struct.

##### *Instances (3):*

File: [2023-01-popcorn/src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L69 )

```solidity
69: mapping(address => mapping(IERC20 => bytes32[])) public userEscrowIdsByToken;
```

File: [2023-01-popcorn/src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L216 )

```solidity
216: mapping(address => mapping(IERC20 => uint256)) public userIndex;
218: mapping(address => mapping(IERC20 => uint256)) public accruedRewards;
```

### <a id=G3>[G-3]</a> Using storage instead of memory for structs/arrays saves gas.

##### Description

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.
Instead of declaring the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct([src](https://ethereum.stackexchange.com/questions/128380/why-using-storage-keyword-instead-of-memory-cost-less-gas))

##### Recommendation

Use storage for `struct/array`

##### *Instances (5):*

File: [2023-01-popcorn/src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L52 )

```solidity
52: Escrow[] memory selectedEscrows = new Escrow[](escrowIds.length);
```

File: [2023-01-popcorn/src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L372 )

```solidity
372: IERC20[] memory _rewardTokens = rewardTokens;
```

File: [2023-01-popcorn/src/vault/VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L155 )

```solidity
155: address[] memory vaultContracts = new address[](1);
156: bytes[] memory rewardsDatas = new bytes[](1);
```

File: [2023-01-popcorn/src/vault/adapter/beefy/BeefyAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L137 )

```solidity
137: address[] memory _rewardTokens = new address[](1);
```

### <a id=G4>[G-4]</a> Usage of `uint`s/`int`s smaller than 32 bytes (256 bits) incurs overhead.

##### Description

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

##### Recommendation

Use `uint256` instead.

##### *Instances (62):*

File: [2023-01-popcorn/src/interfaces/IMultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol#L11 )

```solidity
11: uint32 start;
13: uint32 end;
15: uint32 lastUpdateTime;
36: uint32 duration,
37: uint32 offset
```

File: [2023-01-popcorn/src/interfaces/IMultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L16 )

```solidity
16: uint64 ONE;
21: uint32 rewardsEndTimestamp;
25: uint32 lastUpdatedTimestamp;
32: uint32 escrowDuration;
34: uint32 offset;
44: uint32 escrowDuration,
45: uint32 offset
```

File: [2023-01-popcorn/src/interfaces/vault/IVault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol#L9 )

```solidity
9: uint64 deposit;
10: uint64 withdrawal;
11: uint64 management;
12: uint64 performance;
```

File: [2023-01-popcorn/src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L73 )

```solidity
73: event Locked(IERC20 indexed token, address indexed account, uint256 amount, uint32 duration, uint32 offset);
92: uint32 duration,
93: uint32 offset
114: uint32 start = block.timestamp.safeCastTo32() + offset;
```

File: [2023-01-popcorn/src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L33 )

```solidity
33: uint8 private _decimals;
67: function decimals() public view override(ERC20Upgradeable, IERC20Metadata) returns (uint8) {
171: for (uint8 i; i < _rewardTokens.length; i++) {
220: event RewardInfoUpdate(IERC20 rewardToken, uint160 rewardsPerSecond, uint32 rewardsEndTimestamp);
249: uint32 escrowDuration,
250: uint32 offset
274: uint64 ONE = (10**IERC20Metadata(address(rewardToken)).decimals()).safeCastTo64();
275: uint32 rewardsEndTimestamp = rewardsPerSecond == 0
307: uint32 prevEndTime = rewards.rewardsEndTimestamp;
308: uint32 rewardsEndTimestamp = _calcRewardsEnd(
340: uint32 rewardsEndTimestamp = rewards.rewardsEndTimestamp;
352: uint32 rewardsEndTimestamp,
355: ) internal returns (uint32) {
373: for (uint8 i; i < _rewardTokens.length; i++) {
450: uint8 v,
```

File: [2023-01-popcorn/src/vault/Vault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L38 )

```solidity
38: uint8 internal _decimals;
100: function decimals() public view override returns (uint8) {
669: uint8 v,
```

File: [2023-01-popcorn/src/vault/VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L314 )

```solidity
314: uint8 len = uint8(vaults.length);
319: for (uint8 i = 0; i < len; i++) {
336: uint8 len = uint8(vaults.length);
337: for (uint8 i = 0; i < len; i++) {
353: uint8 len = uint8(vaults.length);
357: for (uint8 i = 0; i < len; i++) {
373: uint8 len = uint8(vaults.length);
374: for (uint8 i = 0; i < len; i++) {
436: uint8 len = uint8(vaults.length);
444: uint24 escrowPercentage,
446: ) = abi.decode(rewardTokenData[i], (address, uint160, uint256, bool, uint224, uint24, uint256));
488: uint8 len = uint8(vaults.length);
517: uint8 len = uint8(vaults.length);
563: uint8 len = uint8(templateCategories.length);
583: uint8 len = uint8(templateCategories.length);
606: uint8 len = uint8(vaults.length);
619: uint8 len = uint8(vaults.length);
632: uint8 len = uint8(vaults.length);
645: uint8 len = uint8(vaults.length);
765: uint8 len = uint8(adapters.length);
805: uint8 len = uint8(adapters.length);
```

File: [2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L38 )

```solidity
38: uint8 internal _decimals;
93: returns (uint8)
637: uint8 v,
```

### <a id=G5>[G-5]</a> `abi.encode()` is less efficient than `abi.encodePacked()`

##### Description

`abi.encode` will apply ABI encoding rules. Therefore all elementary types are padded to 32 bytes and dynamic arrays include their length. Therefore it is possible to also decode this data again (with `abi.decode`) when the type are known.

`abi.encodePacked` will only use the only use the minimal required memory to encode the data. E.g. an address will only use 20 bytes and for dynamic arrays only the elements will be stored without length. For more info see the Solidity docs for packed mode.

For the input of the `keccak` method it is important that you can ensure that the resulting bytes of the encoding are unique. So if you always encode the same types and arrays always have the same length then there is no problem. But if you switch the parameters that you encode or encode multiple dynamic arrays you might have conflicts.
For example:
`abi.encodePacked(address(0x0000000000000000000000000000000000000001), uint(0))` encodes to the same as `abi.encodePacked(uint(0x0000000000000000000000000000000000000001000000000000000000000000), address(0))`
and `abi.encodePacked(uint[](1,2), uint[](3))` encodes to the same as `abi.encodePacked(uint[](1), uint[](2,3))`
Therefore these examples will also generate the same hashes even so they are different inputs.
On the other hand you require less memory and therefore in most cases abi.encodePacked uses less gas than abi.encode.

##### Recommendation

Use `abi.encodePacked()` where possible to save gas

##### *Instances (7):*

File: [2023-01-popcorn/src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L465 )

```solidity
465: abi.encode(
494: abi.encode(
```

File: [2023-01-popcorn/src/vault/Vault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L684 )

```solidity
684: abi.encode(
719: abi.encode(
```

File: [2023-01-popcorn/src/vault/VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L219 )

```solidity
219: abi.encode(asset, address(adminProxy), IStrategy(strategy), harvestCooldown, requiredSigs, strategyData.data),
```

File: [2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L652 )

```solidity
652: abi.encode(
687: abi.encode(
```

### <a id=G6>[G-6]</a> Internal functions only called once can be inlined to save gas.

##### Description

Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.
Check this out: [link](https://betterprogramming.pub/solidity-gas-optimizations-and-tricks-2bcee0f9f1f2)
Note: To sum up, when there is an internal function that is only called once, there is no point for that function to exist.

##### Recommendation

Remove the function and make it inline if it is a simple function.

##### *Instances (15):*

File: [2023-01-popcorn/src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L170 )

```solidity
170: function _getClaimableAmount(Escrow memory escrow) internal view returns (uint256) {
```

File: [2023-01-popcorn/src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L98 )

```solidity
102: function _convertToAssets(uint256 shares, Math.Rounding) internal pure override returns (uint256) {
402: function _accrueRewards(IERC20 _rewardToken, uint256 accrued) internal {
```

File: [2023-01-popcorn/src/vault/VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L154 )

```solidity
154: function _handleVaultStakingRewards(address vault, bytes memory rewardsData) internal {
390: function _registerVault(address vault, VaultMetadata memory metadata) internal {
667: function _verifyCreatorOrOwner(address vault) internal returns (VaultMetadata memory metadata) {
692: function _verifyAdapterConfiguration(address adapter, bytes32 adapterId) internal view {
700: function _verifyEqualArrayLength(uint256 length1, uint256 length2) internal pure {
836: function _setDeploymentController(IDeploymentController _deploymentController) internal {
```

File: [2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L258 )

```solidity
479: function _verifyAndSetupStrategy(bytes4[8] memory requiredSigs) internal {
594: function _protocolDeposit(uint256 assets, uint256 shares) internal virtual {

```

File: [2023-01-popcorn/src/vault/adapter/yearn/YearnAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L80 )

```solidity
89: function _shareValue(uint256 yShares) internal view returns (uint256) {
101: function _freeFunds() internal view returns (uint256) {
109: function _yTotalAssets() internal view returns (uint256) {
114: function _calculateLockedProfit() internal view returns (uint256) {
```

### <a id=G7>[G-7]</a> Multiple accesses of a mapping/array should use a local variable cache.

##### Description

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata. [Similar findings](https://github.com/code-423n4/2022-06-infinity-findings/issues/186)

##### Recommendation

Use a local variable cache.

##### *Instances (4):*

File: [2023-01-popcorn/src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L182 )

```solidity
182: _rewardTokens[i].transfer(user, rewardAmount);
```

File: [2023-01-popcorn/src/vault/PermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol#L43 )

```solidity
43: if (newPermissions[i].endorsed && newPermissions[i].rejected) revert Mismatch();
45: emit PermissionSet(targets[i], newPermissions[i].endorsed, newPermissions[i].rejected);
```

File: [2023-01-popcorn/src/vault/VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L526 )

```solidity
526: rewardTokens[i].transferFrom(msg.sender, address(this), amounts[i]);
```

### <a id=G8>[G-8]</a> `>=` costs less gas than `>`.

##### Description

The compiler uses opcodes `GT` and `ISZERO` for solidity code that uses `>`, but only requires `LT` for `>=`, which [saves 3 gas](https://gist.github.com/IllIllI000/3dc79d25acccfa16dee4e83ffdc6ffde)

##### Recommendation

Use `>=` instead if appropriate.

##### *Instances (38):*

File: [2023-01-popcorn/src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L107 )

```solidity
107: if (feePerc > 0) {
141: return escrows[escrowId].lastUpdateTime != 0 && escrows[escrowId].balance > 0;
```

File: [2023-01-popcorn/src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L178 )

```solidity
178: if (escrowInfo.escrowPercentage > 0) {
255: if (rewards.lastUpdatedTimestamp > 0) revert RewardTokenAlreadyExist(rewardToken);
257: if (amount > 0) {
309: prevEndTime > block.timestamp ? prevEndTime : block.timestamp.safeCastTo32(),
341: if (rewards.rewardsPerSecond > 0) {
356: if (rewardsEndTimestamp > block.timestamp)
377: if (rewards.rewardsPerSecond > 0) _accrueRewards(rewardToken, _accrueStatic(rewards));
392: if (rewards.rewardsEndTimestamp > block.timestamp) {
394: } else if (rewards.rewardsEndTimestamp > rewards.lastUpdatedTimestamp) {
```

File: [2023-01-popcorn/src/vault/Vault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L149 )

```solidity
149: if (feeShares > 0) _mint(feeRecipient, feeShares);
189: if (feeShares > 0) _mint(feeRecipient, feeShares);
235: if (feeShares > 0) _mint(feeRecipient, feeShares);
273: if (feeShares > 0) _mint(feeRecipient, feeShares);
432: managementFee > 0
453: performanceFee > 0 && shareValue > highWaterMark
486: if (shareValue > highWaterMark) highWaterMark = shareValue;
488: if (managementFee > 0) feesUpdatedAt = block.timestamp;
490: if (totalFee > 0 && currentAssets > 0)
630: if (_quitPeriod < 1 days || _quitPeriod > 7 days)
```

File: [2023-01-popcorn/src/vault/VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L103 )

```solidity
103: if (adapterData.id > 0)
112: if (rewardsData.length > 0) _handleVaultStakingRewards(vault, rewardsData);
169: if (initialDeposit > 0) {
211: if (strategyData.id > 0) {
694: if (adapterId > 0) revert AdapterConfigFaulty();
753: if (newFee > 2e17) revert InvalidPerformanceFee(newFee);
793: if (newCooldown > 1 days) revert InvalidHarvestCooldown(newCooldown);
```

File: [2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L116 )

```solidity
116: if (assets > maxDeposit(receiver)) revert MaxError(assets);
135: if (shares > maxMint(receiver)) revert MaxError(shares);
178: if (assets > maxWithdraw(owner)) revert MaxError(assets);
198: if (shares > maxRedeem(owner)) revert MaxError(shares);
535: performanceFee_ > 0 && shareValue > highWaterMark_
551: if (newFee > 2e17) revert InvalidPerformanceFee(newFee);
563: if (fee > 0) _mint(FEE_RECIPIENT, convertToShares(fee));
566: if (shareValue > highWaterMark) highWaterMark = shareValue;
```

File: [2023-01-popcorn/src/vault/adapter/beefy/BeefyAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L156 )

```solidity
156: if (beefyFee > 0)
177: if (beefyFee > 0)
```

### <a id=G9>[G-9]</a> `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

##### Description

Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**

##### Recommendation

Use `<x> = <x> + <y>` instead.

##### *Instances (12):*

File: [2023-01-popcorn/src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L110 )

```solidity
110: amount -= fee;
162: escrows[escrowId].balance -= claimable;
```

File: [2023-01-popcorn/src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L357 )

```solidity
357: amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);
408: rewardInfos[_rewardToken].index += deltaIndex;
431: accruedRewards[_user][_rewardToken] += supplierDelta;
```

File: [2023-01-popcorn/src/vault/Vault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L228 )

```solidity
228: shares += feeShares;
343: shares += shares.mulDiv(
365: assets += assets.mulDiv(
387: assets -= assets.mulDiv(
```

File: [2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L158 )

```solidity
158: underlyingBalance += _underlyingBalance() - underlyingBalance_;
225: underlyingBalance -= underlyingBalance_ - _underlyingBalance();
```

File: [2023-01-popcorn/src/vault/adapter/beefy/BeefyAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L178 )

```solidity
178: assets -= assets.mulDiv(
```
