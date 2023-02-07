# Local variable name shadows global name
The variable `owner` is used in both these code snippets:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L121-L134
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L445-L485
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L57-L98
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L211-L240
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L253-L278
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L664-L707

and it shadows the global `owner` variable from the `OwnedUpgradeable` contract being inherited. You should be careful about this and rename it to something like `user` or `_owner`, depending on the context, that is less ambiguous.

# Add check that reward token is not the contract itself

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L243-L253

There is a small chance this will actually happen, but just in case, to avoid the weird outcomes that come with having a reward token that is also the vault itself like compound effects, we should add a new revert statement:
```
    if (address(this) == address(rewardToken)) revert RewardTokenCantBeSelf();
```