# Summary
|ID     | Finding|  Gas saved |Instances|
|:----: | :---           |  :----:         |:----:         |
|G-01       |Make for loop unchecked|-|-|
|G-02       |It's not necessary to do a safecast on block.timestamp|450|9|
|G-03       |Use unchecked block when computation can't under/overflow|-|2|
|G-04       |Checks are unnecessary in `_transfer` function |240|2|
|G-05       |Don't assign to a memory variable when it's only used once |783|1|


# Details
## [G-01] Make for loop unchecked
The risk of for loops getting overflowed is extremely low. Because it always increments by 1 and is limited to the arrays length. Even if the arrays are extremely long, it will take a massive amount of time and gas to let the for loop overflow. The longer the array, the more gas you will save.

```diff
-   for (uint256 i = 0; i < escrowIds.length; i++) {
+   for (uint256 i = 0; i < escrowIds.length; ) {
      selectedEscrows[i] = escrows[escrowIds[i]];
+     unchecked{
+        i++;
+      }
    }
```

- [MultiRewardEscrow.sol#L53-L55](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L53-L55): `getEscrows()` gas saved: 105
- [MultiRewardEscrow.sol#L155-L167](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L155-L167): `claimRewards()` gas saved: 75
- [MultiRewardEscrow.sol#L210-L215](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L210-L215): `setFees()` gas saved: 68
The same amount of gas can be saved for all the for loops with uint256 in [VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol) and [PermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol)

## [G-02] It's not necessary to do a safecast on block.timestamp
When you cast block.timestamp to a uint32 it will never overflow. That's why it's not necessary to safecast. You can cast to uint32 in the regular way instead. Saves around 50 gas.
```diff
-    block.timestamp.safeCastTo32()
+    uint32(block.timestamp)
```
- [MultiRewardEscrow.sol#L114](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L114)
- [MultiRewardEscrow.sol#L163](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L163)
- [MultiRewardEscrow.sol#L175](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L175)
- [MultiRewardStaking.sol#L276](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L276)
- [MultiRewardStaking.sol#L284](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L284)
- [MultiRewardStaking.sol#L309](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L309)
- [MultiRewardStaking.sol#L346](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L346)
- [MultiRewardStaking.sol#L409](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L409)

## [G-03] Use unchecked block when computation can't under/overflow
When a variable is incremented by one there is no risk in using a unchecked block. It will take a massive amount of time and gas to make it overflow
- [MultiRewardEscrow.sol#L102](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L102)

An unchecked block can also be used when previous functions or statement make sure there is no possibility of over/underflowing
- [MultiRewardEscrow.sol#L162](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L162)

## [G-04] Checks are unnecessary in `_transfer` function
All the if statements are unnecessary because they are also in the `_mint` and `_burn` functions. It's just a gas saver to do them twice

[MultiRewardStaking.sol#L147-L148](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L147-L148)
- `transfer()` gas saved: 241
- `transferFrom()` gas saved: 242
## [G-05] Don't assign to a memory variable when it's only used once
When a variable is only accessed once it's a waste of gas to assign it to a memory variable first. Especially when you write struct to memory like in the following example, you can save a lot of gas with this method.

[MultiRewardStaking.sol#L255](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L255):`addRewardToken()` gas saved: 783
```diff
-    RewardInfo memory rewards = rewardInfos[rewardToken];
-    if (rewards.lastUpdatedTimestamp > 0) revert RewardTokenAlreadyExist(rewardToken);
+    if (rewardInfos[rewardToken].lastUpdatedTimestamp > 0) revert RewardTokenAlreadyExist(rewardToken);
```