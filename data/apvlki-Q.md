1. Adding initial staking rewards token and funding staking rewards will revert if the `rewardToken` does not revert.

The affected functions are the following:
- [VaultController.addStakingRewardsToken()](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L433)
- [VaultController.fundStakingRewards()](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L512)

In instances that reward tokens are unable to revert, the contract will proceed to call the `MultiRewardStaking` contract right away. Luckily, the transfer used in `MultiRewardStaking.addRewardToken()` and  `MultiRewardStaking.fundReward()` contains `safeTransferFrom` which will revert if no tokens were transferred to the `adminProxy` and `VaultController`.

2. Possibility of `MultiRewardEscrow.claimReward()` to be vulnerable to a reentrancy attack

There are a bunch of external calls before setting `accruedRewards[user][_rewardTokens[i]] `to zero. Malicious actors can add some exploits on the external calls potentially draining the rewards pool of that reward token. It is recommended to refactor this conforming to the check-effects pattern

3. Setting max allowance during token transfers

Setting max allowance might completely drain user funds if exploited by malicious actors. 

This was observed in the following contracts:
- `Vault.sol`
- `VaultController.sol`
- `AdapterBase.sol`
- `BeefyAdapter.sol`
- `MultiRewardsStaking.sol`


