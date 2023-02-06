## Use of safeapprove()

### Line Of Code
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L271 

### Details & Impact

The safeapprove() method is used,instead of  safeIncreaseAllowance() and safeDecreaseAllowance(). I however argue that this isn’t recommended because:
* OpenZeppelin’s documentation discourages the use of safeapprove(), use safeIncreaseAllowance() and safeDecreaseAllowance() whenever possible.

### Tools Used

Manual code review