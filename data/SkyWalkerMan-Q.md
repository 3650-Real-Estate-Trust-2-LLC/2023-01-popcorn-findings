### Use ***TransferFrom*** instead of ***safeTransferFrom***.
***safeTransferFrom*** is an ***internal*** function.
here it's used for ***external*** function:
```
File: src\utils\MultiRewardStaking.sol

function addRewardToken(
    IERC20 rewardToken,
    uint160 rewardsPerSecond,
    uint256 amount,
    bool useEscrow,
    uint192 escrowPercentage,
    uint32 escrowDuration,
    uint32 offset
  ) external onlyOwner {
    if (asset() == address(rewardToken)) revert RewardTokenCantBeStakingToken();

    RewardInfo memory rewards = rewardInfos[rewardToken];
    if (rewards.lastUpdatedTimestamp > 0) revert RewardTokenAlreadyExist(rewardToken);

    if (amount > 0) {
     if (rewardsPerSecond == 0) revert ZeroRewardsSpeed();
259: rewardToken.safeTransferFrom(msg.sender, address(this), amount);
    }
}
```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L259)