# 1. Lines too long

In general, it is a good practice to keep lines of source code within 80 characters in length.
Although, some flexibility is allowed and it is reasonable to let lines be up to 120 characters in some instances. 

On modern screens, it is even possible to go beyond this limit.
However, it is recommended to split lines when they reach a length of 164 characters or more, as this is the point at which GitHub will introduce a scroll bar to view the code.

This can help to make the code more readable and easier to work with.

Affected line of code:

- [AdapterBase.sol#L6](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L6)
- [MultiRewardStaking.sol#L7](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L7)

# 2. Use the `delete` operator to clear variables, rather than assigning a value of `false`.

To clear variables, consider using the `delete` operator rather than assigning to `false`, because this conveys the intention more clearly and is more idiomatic.

As an example on [line 186](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L186) you can refactor the code like so:

```solidity
Line 186:    delete accruedRewards[user][_rewardTokens[i]];
```

Affected line of code:

- [MultiRewardStaking.sol#L186](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L186)

- [AdapterBase.sol#L577](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L577)

- [TemplateRegistry.sol#L75](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L75)

# 3. Open TODOs

Open TODOs can point to architecture or programming issues that still need to be resolved. Consider resolving them before deploying.

Affected line of code:

- [AdapterBase.sol#L516](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L516)

# 4. Use scientific notation (`1e18`) rather than exponential (`10**18`)

Improves readability.

Affected line of code:

- [YearnAdapter.sol#L25](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L25)
