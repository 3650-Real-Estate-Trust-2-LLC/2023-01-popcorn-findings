## [G-01] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

### Description

Gas consumption can be greater if you use items that are less than 32 bytes in size. This is such that the EVM can only handle 32 bytes at once. In order to increase the element's size from 32 bytes to the necessary amount, the EVM must do extra operations if it is lower than that. When necessary, it is advised to utilize a bigger size and then downcast.

### Findings

[src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol)
```Solidity
::114 =>     uint32 start = block.timestamp.safeCastTo32() + offset;
```
[src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol)
```Solidity
::274 =>     uint64 ONE = (10**IERC20Metadata(address(rewardToken)).decimals()).safeCastTo64();
::275 =>     uint32 rewardsEndTimestamp = rewardsPerSecond == 0
::307 =>     uint32 prevEndTime = rewards.rewardsEndTimestamp;
::308 =>     uint32 rewardsEndTimestamp = _calcRewardsEnd(
::340 =>     uint32 rewardsEndTimestamp = rewards.rewardsEndTimestamp;
```
[src/vault/VaultController.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol)
```Solidity
::314 =>     uint8 len = uint8(vaults.length);
::319 =>     for (uint8 i = 0; i < len; i++) {
::336 =>     uint8 len = uint8(vaults.length);
::337 =>     for (uint8 i = 0; i < len; i++) {
::353 =>     uint8 len = uint8(vaults.length);
::357 =>     for (uint8 i = 0; i < len; i++) {
::373 =>     uint8 len = uint8(vaults.length);
::374 =>     for (uint8 i = 0; i < len; i++) {
::436 =>     uint8 len = uint8(vaults.length);
::446 =>       ) = abi.decode(rewardTokenData[i], (address, uint160, uint256, bool, uint224, uint24, uint256));
::488 =>     uint8 len = uint8(vaults.length);
::517 =>     uint8 len = uint8(vaults.length);
::563 =>     uint8 len = uint8(templateCategories.length);
::583 =>     uint8 len = uint8(templateCategories.length);
::606 =>     uint8 len = uint8(vaults.length);
::619 =>     uint8 len = uint8(vaults.length);
::632 =>     uint8 len = uint8(vaults.length);
::645 =>     uint8 len = uint8(vaults.length);
::765 =>     uint8 len = uint8(adapters.length);
::805 =>     uint8 len = uint8(adapters.length);
```

### References

- [Layout of State Variables in Storage | Solidity docs](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html#layout-of-state-variables-in-storage)
- [GAS OPTIMIZATIONS ISSUES by Gokberk Gulgun](https://hackmd.io/@W1m6lTsFT5WAy9C_lRTX_g/rkr5Laoys)

## [G-02] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow

### Description

In solidity version 0.8.0 and higher, unchecked saves 30-40 gas per loop

### Findings

[src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol)
```Solidity
::53 =>     for (uint256 i = 0; i < escrowIds.length; i++) {
::155 =>     for (uint256 i = 0; i < escrowIds.length; i++) {
::210 =>     for (uint256 i = 0; i < tokens.length; i++) {
```
[src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol)
```Solidity
::171 =>     for (uint8 i; i < _rewardTokens.length; i++) {
::373 =>     for (uint8 i; i < _rewardTokens.length; i++) {  
```
[src/vault/PermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol)
```Solidity
::42 =>     for (uint256 i = 0; i < len; i++) {
```
[src/vault/VaultController.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol)
```Solidity
::319 =>     for (uint8 i = 0; i < len; i++) {
::337 =>     for (uint8 i = 0; i < len; i++) {
::357 =>     for (uint8 i = 0; i < len; i++) {
::374 =>     for (uint8 i = 0; i < len; i++) {
::437 =>     for (uint256 i = 0; i < len; i++) {
::494 =>     for (uint256 i = 0; i < len; i++) {
::523 =>     for (uint256 i = 0; i < len; i++) {
::564 =>     for (uint256 i = 0; i < len; i++) {
::587 =>     for (uint256 i = 0; i < len; i++) {
::607 =>     for (uint256 i = 0; i < len; i++) {
::620 =>     for (uint256 i = 0; i < len; i++) {
::633 =>     for (uint256 i = 0; i < len; i++) {
::646 =>     for (uint256 i = 0; i < len; i++) {
::766 =>     for (uint256 i = 0; i < len; i++) {
::806 =>     for (uint256 i = 0; i < len; i++) {
```

### References

- [[G-02] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS](https://code4rena.com/reports/2022-12-caviar/#g-02-ii-should-be-uncheckediuncheckedi-when-it-is-not-possible-for-them-to-overflow-as-is-the-case-when-used-in-for--and-while-loops)

## [G-03] Superfluos event fields

### Description

In the event information, `block.number` and `block.timestamp` are already added by default.

### Findings

[src/vault/Vault.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)
```Solidity
::512 =>     event NewFeesProposed(VaultFees newFees, uint256 timestamp);
::569 =>     event NewAdapterProposed(IERC4626 newAdapter, uint256 timestamp);
```

### Resources

- [[G-08] SUPERFLUOUS EVENT FIELDS](https://code4rena.com/reports/2022-12-caviar/#g-08-superfluous-event-fields)