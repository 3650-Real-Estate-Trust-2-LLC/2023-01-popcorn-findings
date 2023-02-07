# Report
## Gas Optimizations ##
### [G-1]: The increment in for loop post condition can be made unchecked
**Context:**

1. ```for (uint256 i = 0; i < len; i++) {``` [L42](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L42) 
1. ```for (uint256 i = 0; i < escrowIds.length; i++) {``` [L53](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L53) 
1. ```nonce++;``` [L102](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L102) 
1. ```for (uint256 i = 0; i < escrowIds.length; i++) {``` [L155](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L155) 
1. ```for (uint256 i = 0; i < tokens.length; i++) {``` [L210](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L210) 
1. ```for (uint8 i; i < _rewardTokens.length; i++) {``` [L171](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L171) 
1. ```for (uint8 i; i < _rewardTokens.length; i++) {``` [L373](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L373) 
1. ```for (uint8 i = 0; i < len; i++) {``` [L319](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L319) 
1. ```for (uint8 i = 0; i < len; i++) {``` [L337](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L337) 
1. ```for (uint8 i = 0; i < len; i++) {``` [L357](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L357) 
1. ```for (uint8 i = 0; i < len; i++) {``` [L374](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L374) 
1. ```for (uint256 i = 0; i < len; i++) {``` [L437](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L437) 
1. ```for (uint256 i = 0; i < len; i++) {``` [L494](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L494) 
1. ```for (uint256 i = 0; i < len; i++) {``` [L523](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L523) 
1. ```for (uint256 i = 0; i < len; i++) {``` [L564](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L564) 
1. ```for (uint256 i = 0; i < len; i++) {``` [L587](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L587) 
1. ```for (uint256 i = 0; i < len; i++) {``` [L607](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L607) 
1. ```for (uint256 i = 0; i < len; i++) {``` [L620](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L620) 
1. ```for (uint256 i = 0; i < len; i++) {``` [L633](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L633) 
1. ```for (uint256 i = 0; i < len; i++) {``` [L646](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L646) 
1. ```for (uint256 i = 0; i < len; i++) {``` [L766](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L766) 
1. ```for (uint256 i = 0; i < len; i++) {``` [L806](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L806) 

**Description:**

[This](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked) can save 30-40 gas per loop iteration.

**Recommendation:**

Example how to fix. Change:
```
for (uint256 i = 0; i < orders.length; ++i) {
   // Do the thing
}
```

To:
```
for (uint256 i = 0; i < orders.length;) {
   // Do the thing
   unchecked { ++i; }
}
```

### [G-2]: Place subtractions where the operands can't underflow in unchecked {} block
**Context:**

1. ```return _depositLimit - assets;``` [L151](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L151) 
1. ```amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);``` [L357](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L357) (rewardsEndTimestamp - block.timestamp)
1. ```elapsed = rewards.rewardsEndTimestamp - rewards.lastUpdatedTimestamp;``` [L395](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L395) 
1. ```(shareValue - highWaterMark) * totalSupply(),``` [L455](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L455) (shareValue - highWaterMark)
1. ```(shareValue - highWaterMark_) * totalSupply(),``` [L537](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L537) (shareValue - highWaterMark_)

**Description:**

Some gas can be saved by using an unchecked {} block if an underflow isn't possible because of a previous require() or if-statement.

### [G-3]: Storage pointer 
**Context:**

1. ```Escrow memory escrow = escrows[escrowId];``` [L157](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L157) 
1. ```EscrowInfo memory escrowInfo = escrowInfos[_rewardTokens[i]];``` [L176](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L176) 
1. ```RewardInfo memory rewards = rewardInfos[rewardToken];``` [L254](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L254) 
1. ```RewardInfo memory rewards = rewardInfos[rewardToken];``` [L297](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L297) 
1. ```RewardInfo memory rewards = rewardInfos[rewardToken];``` [L328](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L328) 
1. ```RewardInfo memory rewards = rewardInfos[rewardToken];``` [L375](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L375) 
1. ```RewardInfo memory rewards = rewardInfos[_rewardToken];``` [L414](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L414) 

**Recommendation:**

Every time you copy a storage struct/array/mapping to a memory variable, you are literally copying each member by reading it from storage, which is expensive. And when you use the storage keyword, you are just storing a pointer to the storage, which is much cheaper.
