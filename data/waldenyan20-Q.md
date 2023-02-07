# Local variable name shadows global name
The variable `owner` is used in both these code snippets:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L121-L134
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L445-L485

and it shadows the global `owner` variable from the `OwnedUpgradeable` contract being inherited. You should be careful about this and rename it to something like `user` that is less ambiguous.