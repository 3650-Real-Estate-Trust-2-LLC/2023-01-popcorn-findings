
---------

**Gas optimization**

------------

| S No. | Issue | Instances | Gas Savings (from provided tests) |
|-----|-----|-----|-----|
| [G-01] | Increments can be unchecked | 12 | 480
| [G-02] | Usage of uints or ints smaller than 32 bytes incurs overhead | 31 | 868
| [G-03] | x += y costs more gas than x = x + y for state variables (same applies for x -= y) | 12 | 1356
| [G-04] | Internal functions only called once can be inlined to save gas | 14 | 560
| [G-05] | Using both named returns and a return statement isn't necessary | 2 | 6
| [G-06] | Use a more recent version of solidity | 35 | -

----------

## G-01 Increments can be unchecked

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline

Prior to Solidity 0.8.0, arithmetic operations would always wrap in case of under- or overflow leading to widespread use of libraries that introduce additional checks.

Since Solidity 0.8.0, all arithmetic operations revert on over- and underflow by default, thus making the use of these libraries unnecessary.

To obtain the previous behaviour, an unchecked block can be used. It saves **30-40 gas** per loop.

_There is 12 instances of this issue_

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol

```
File: src/utils/MultiRewardEscrow.sol

53: for (uint256 i = 0; i < escrowIds.length; i++) {
155: for (uint256 i = 0; i < escrowIds.length; i++) {
210: for (uint256 i = 0; i < tokens.length; i++) {
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol

```
File: src/utils/MultiRewardStaking.sol

171: for (uint8 i; i < _rewardTokens.length; i++) {
373: for (uint8 i; i < _rewardTokens.length; i++) {
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol

```
File: src/vault/PermissionRegistry.sol

42: for (uint256 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol

```
File: src/vault/VaultController.sol

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

-------

## G-02 Usage of uints or ints smaller than 32 bytes incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html) Each operation involving a `uint8` costs an extra [**22-28 gas**](https://gist.github.com/IllIllI000/9388d20c70f9a4632eb3ca7836f54977) (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

_There are 31 instances of this issue:_

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVaultController.sol

```
File: src/interfaces/vault/IVaultController.sol

60: uint160[] memory rewardsSpeeds
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol

```
File: src/utils/MultiRewardEscrow.sol

114: uint32 start = block.timestamp.safeCastTo32() + offset;
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol

```
File: src/utils/MultiRewardStaking.sol

33: uint8 private _decimals;
245: uint160 rewardsPerSecond,
340: uint32 rewardsEndTimestamp = rewards.rewardsEndTimestamp;
450: uint8 v,
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol

```
File: src/vault/Vault.sol

38: uint8 internal _decimals;
669: uint8 v,
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol

```
File: src/vault/VaultController.sol

314: uint8 len = uint8(vaults.length);
319: for (uint8 i = 0; i < len; i++) {
336: uint8 len = uint8(vaults.length);
337: for (uint8 i = 0; i < len; i++) {
353: uint8 len = uint8(vaults.length);
357: for (uint8 i = 0; i < len; i++) {
373: uint8 len = uint8(vaults.length);
374: for (uint8 i = 0; i < len; i++) {
436: uint8 len = uint8(vaults.length);
437: for (uint256 i = 0; i < len; i++) {
440: uint160 rewardsPerSecond,
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

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol

```
File: src/vault/adapter/abstracts/AdapterBase.sol

38: uint8 internal _decimals;
637: uint8 v,
```

----------

## G-03 x += y costs more gas than x = x + y for state variables (same applies for x -= y)

Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**.

_There are 12 instances of this issue_

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol

```
File: src/utils/MultiRewardEscrow.sol

110: amount -= fee;
162: escrows[escrowId].balance -= claimable;
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol

```
File: src/utils/MultiRewardStaking.sol

357: amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);
408: rewardInfos[_rewardToken].index += deltaIndex;
431: accruedRewards[_user][_rewardToken] += supplierDelta;
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol

```
File: src/vault/Vault.sol

228: shares += feeShares;
343: shares += shares.mulDiv
365: assets += assets.mulDiv(
387: assets -= assets.mulDiv(
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol

```
File: src/vault/adapter/abstracts/AdapterBase.sol

158: underlyingBalance += _underlyingBalance() - underlyingBalance_;
225: underlyingBalance -= underlyingBalance_ - _underlyingBalance();
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol

```
File: src/vault/adapter/beefy/BeefyAdapter.sol

178: assets -= assets.mulDiv(
```

----------

## G-04 Internal functions only called once can be inlined to save gas

Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls.

_There are 14 instances of this issue_

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol

```
File: src/utils/MultiRewardStaking.sol

191: function _lockToken(
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol

```
File: src/vault/VaultController.sol

120: function _deployVault(VaultInitParams memory vaultData, IDeploymentController _deploymentController)
141: function _registerCreatedVault(
154: function _handleVaultStakingRewards(address vault, bytes memory rewardsData) internal {
225: function __deployAdapter(
242: function _encodeAdapterData(DeploymentArgs memory adapterData, bytes memory baseAdapterData)
256: function _deployStrategy(DeploymentArgs memory strategyData, IDeploymentController _deploymentController)
390: function _registerVault(address vault, VaultMetadata memory metadata) internal {
692: function _verifyAdapterConfiguration(address adapter, bytes32 adapterId) internal view {
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol

```
File: src/vault/adapter/abstracts/AdapterBase.sol

479: function _verifyAndSetupStrategy(bytes4[8] memory requiredSigs) internal {
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol

```
File: src/vault/adapter/yearn/YearnAdapter.sol

89: function _shareValue(uint256 yShares) internal view returns (uint256) {
101: function _freeFunds() internal view returns (uint256) {
109: function _yTotalAssets() internal view returns (uint256) {
114: function _calculateLockedProfit() internal view returns (uint256) {
```

-----

## G-05 Using both named returns and a return statement isn't necessary 

Removing unused named returns variables can reduce gas usage (MSTOREs/MLOADs) and improve code clarity. To save gas and improve code quality: consider using only one of those.

Each MLOAD and MSTORE costs **3 additional gas**

_There are 2 instances of this issue_

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol

```
File: src/vault/AdminProxy.sol

19-22: function execute(address target, bytes calldata callData)
    external
    onlyOwner
    returns (bool success, bytes memory returndata)
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol

```
File: src/vault/adapter/abstracts/AdapterBase.sol

380-385: function _convertToShares(uint256 assets, Math.Rounding rounding)
        internal
        view
        virtual
        override
        returns (uint256 shares)
```

-------

## G-06 Use a more recent version of solidity

_This issue exists in all the In-scope contracts_. _There are 35 instances of this issue_

[EIP165.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/EIP165.sol#L4) , [OnlyStrategy.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/OnlyStrategy.sol#L4) , [AdminProxy.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L4) , [WithRewards.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/WithRewards.sol#L4) , [CloneFactory.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneFactory.sol#L4) , [PermissionRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L4) , [CloneRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L4) , [VaultRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultRegistry.sol#L4) , [DeploymentController.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L4) , [TemplateRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L4) , [YearnAdapter.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L4) , [MultiRewardEscrow.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L4) , [BeefyAdapter.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L4) , [MultiRewardStaking.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L4) , [Vault.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L4) , [VaultController.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L3) , [AdapterBase.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L4) , [IEIP165.sol#L2](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IEIP165.sol#L2) , [IAdminProxy.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IAdminProxy.sol#L4) , [IWithRewards.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IWithRewards.sol#L4) , [ICloneFactory.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ICloneFactory.sol#L4) , [IStrategy.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IStrategy.sol#L4) , [ICloneRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ICloneRegistry.sol#L4) , [IPermissionRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IPermissionRegistry.sol#L4) , [IVaultRegistry.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVaultRegistry.sol#L3) , [IYearn.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/IYearn.sol#L4) , [IAdapter.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IAdapter.sol#L4) , [IDeploymentController.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IDeploymentController.sol#L4) , [ITemplateRegistry.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ITemplateRegistry.sol#L3) , [IMultiRewardEscrow.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardEscrow.sol#L3) , [IERC4626.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IERC4626.sol#L3) , [IBeefy.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/IBeefy.sol#L4) , [IMultiRewardStaking.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L3) , [IVault.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVault.sol#L3) , [IVaultController.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVaultController.sol#L4)

-------


