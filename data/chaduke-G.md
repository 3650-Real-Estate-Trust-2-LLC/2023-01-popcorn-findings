G1. We can save gas by eliminating the multiplication and one line for the case ``rewardsEndTimestamp > block.timestamp``:
```diff
 function _calcRewardsEnd(
    uint32 rewardsEndTimestamp,
    uint160 rewardsPerSecond,
    uint256 amount
  ) internal returns (uint32) {
    if (rewardsEndTimestamp > block.timestamp)
-       amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);
+      return rewardsEndTimestamp + (amount / uint256(rewardsPerSecond))).safeCastTo32();

    return (block.timestamp + (amount / uint256(rewardsPerSecond))).safeCastTo32();
  }
```