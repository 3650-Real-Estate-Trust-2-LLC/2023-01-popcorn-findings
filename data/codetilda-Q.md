## [L-01] Missing ReEntrancy Guard to *claimRewards* function in *MultiRewardStaking.sol*

Violation of the CEI(Checks/Effects/Iteraction) pattern without ReEntrancy guard may result in an unsafe external call (ReEntrancy Attack).

Instance:

[MultiRewardStaking.sol#L170](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L170)

  
    function claimRewards(address user, IERC20[] memory _rewardTokens) external accrueRewards(msg.sender, user) {
      for (uint8 i; i < _rewardTokens.length; i++) {
      
      @audit CHECKS
      uint256 rewardAmount = accruedRewards[user][_rewardTokens[i]];

      if (rewardAmount == 0) revert ZeroRewards(_rewardTokens[i]);
          
      EscrowInfo memory escrowInfo = escrowInfos[_rewardTokens[i]];
      
      @audit ITERACTION
      if (escrowInfo.escrowPercentage > 0) {
        _lockToken(user, _rewardTokens[i], rewardAmount, escrowInfo);
        emit RewardsClaimed(user, _rewardTokens[i], rewardAmount, true);
      } else {
        _rewardTokens[i].transfer(user, rewardAmount);
        emit RewardsClaimed(user, _rewardTokens[i], rewardAmount, false);
      }

      @audit EFFECTS
      accruedRewards[user][_rewardTokens[i]] = 0;
      }
    }


### Recommended Mitigation Steps

Move line of code *accruedRewards[user][_rewardTokens[i]] = 0;* to the top before actual "Iteraction" part of function to achieve CEI pattern

    function claimRewards(address user, IERC20[] memory _rewardTokens) external accrueRewards(msg.sender, user) {
      for (uint8 i; i < _rewardTokens.length; i++) {
      
      @audit CHECKS
      uint256 rewardAmount = accruedRewards[user][_rewardTokens[i]];

      if (rewardAmount == 0) revert ZeroRewards(_rewardTokens[i]);
          
      EscrowInfo memory escrowInfo = escrowInfos[_rewardTokens[i]];
      
      + @audit EFFECTS
      + accruedRewards[user][_rewardTokens[i]] = 0;
      
      @audit ITERACTION
      if (escrowInfo.escrowPercentage > 0) {
        _lockToken(user, _rewardTokens[i], rewardAmount, escrowInfo);
        emit RewardsClaimed(user, _rewardTokens[i], rewardAmount, true);
      } else {
        _rewardTokens[i].transfer(user, rewardAmount);
        emit RewardsClaimed(user, _rewardTokens[i], rewardAmount, false);
      }

      - @audit EFFECTS
      - accruedRewards[user][_rewardTokens[i]] = 0;

      }
    }

