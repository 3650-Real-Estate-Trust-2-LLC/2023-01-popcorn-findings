
| S No. | Issue | Instances | Gas Savings|
|-----|-----|-----|-----|
| [G-01] | ++I or I++ SHOULD BE UNCHECKED{++I} or UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS | 12 | 480 
| [G-02] | X += Y COSTS MORE GAS THAN X = X + Y FOR STATE VARIABLES | 12 | 1356
| [G-03] | USING BOTH NAMED RETURNS AND A RETURN STATEMENT ISNT NECESSARY | 2 | 12
| [G-04] | USAGE OF UINTS or INTS SMALLER THAN 32 BYTES - 256 BITS INCURS OVERHEAD | 31 | 868 
| [G-05] | INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS | 14 | 560 




### G-01 ++I or I++ SHOULD BE UNCHECKED{++I} or UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS

*Number of Instances Identified: 12*

The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**.

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol

```
53: for (uint256 i = 0; i < escrowIds.length; i++) {
155: for (uint256 i = 0; i < escrowIds.length; i++) {
210: for (uint256 i = 0; i < tokens.length; i++) {
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol

```
171: for (uint8 i; i < _rewardTokens.length; i++)
373: for (uint8 i; i < _rewardTokens.length; i++)
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol

```
42: for (uint256 i = 0; i < len; i++)
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol

```
319: for (uint8 i = 0; i < len; i++)
337: for (uint8 i = 0; i < len; i++)
357: for (uint8 i = 0; i < len; i++)
374: for (uint8 i = 0; i < len; i++)
437: for (uint256 i = 0; i < len; i++)
494: for (uint256 i = 0; i < len; i++)
523: for (uint256 i = 0; i < len; i++)
564: for (uint256 i = 0; i < len; i++)
587: for (uint256 i = 0; i < len; i++)
607: for (uint256 i = 0; i < len; i++)
620: for (uint256 i = 0; i < len; i++)
633: for (uint256 i = 0; i < len; i++)
646: for (uint256 i = 0; i < len; i++)
766: for (uint256 i = 0; i < len; i++)
806: for (uint256 i = 0; i < len; i++)
```


### G-02 X += Y COSTS MORE GAS THAN X = X + Y FOR STATE VARIABLES

*Number of Instances Identified: 12*

Using the addition operator instead of plus-equals saves **[113 gas]([https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**).

_There are 12 instances of this issue_

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol

```
110: amount -= fee;
162: escrows[escrowId].balance -= claimable;
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol

```
357: amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);
408: rewardInfos[_rewardToken].index += deltaIndex;
431: accruedRewards[_user][_rewardToken] += supplierDelta;
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol

```
228: shares += feeShares;
343: shares += shares.mulDiv
365: assets += assets.mulDiv(
387: assets -= assets.mulDiv(
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol

```
158: underlyingBalance += _underlyingBalance() - underlyingBalance_;
225: underlyingBalance -= underlyingBalance_ - _underlyingBalance();
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol

```
178: assets -= assets.mulDiv(
```


### G-03 USING BOTH NAMED RETURNS AND A RETURN STATEMENT ISNT NECESSARY

*Number of Instances Identified: 2*

Removing unused named returns variables can reduce gas usage (MSTOREs/MLOADs) and improve code clarity. To save gas and improve code quality: consider using only one of those.
Each MLOAD and MSTORE costs `3 additional gas`

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol

```
19-22: function execute(address target, bytes calldata callData)
    external
    onlyOwner
    returns (bool success, bytes memory returndata)
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol

```
380-385: function _convertToShares(uint256 assets, Math.Rounding rounding)
        internal
        view
        virtual
        override
        returns (uint256 shares)
```



### G-04 USAGE OF UINTS or INTS SMALLER THAN 32 BYTES - 256 BITS INCURS OVERHEAD

*Number of Instances Identified: 31*

> When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html) Use a larger size then downcast where needed

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVaultController.sol

```
60: uint160[] memory rewardsSpeeds
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol

```
114: uint32 start = block.timestamp.safeCastTo32() + offset;
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol

```
33: uint8 private _decimals;
245: uint160 rewardsPerSecond,
340: uint32 rewardsEndTimestamp = rewards.rewardsEndTimestamp;
450: uint8 v,
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol

```
38: uint8 internal _decimals;
669: uint8 v,
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol

```
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
38: uint8 internal _decimals;
637: uint8 v,
```


### G-05 INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS

*Number of Instances Identified: 14*

Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls.


https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol

```
191: function _lockToken()
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol

```
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
479: function _verifyAndSetupStrategy(bytes4[8] memory requiredSigs) internal {
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol

```
89: function _shareValue(uint256 yShares) internal view returns (uint256) {
101: function _freeFunds() internal view returns (uint256) {
109: function _yTotalAssets() internal view returns (uint256) {
114: function _calculateLockedProfit() internal view returns (uint256) {
```


