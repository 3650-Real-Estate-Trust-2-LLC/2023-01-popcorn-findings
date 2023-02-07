Instead of implementing custom `Owned` contract the Open Zeppelin's Ownable2Step could be used:
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol

Note that Open Zeppelin has Ownable2StepUpgradeable version too.

***

Consider using the type-aware `abi.encodeCall` method instead of  `abi.encodeWithSignature` and `abi.encodeWithSelector`.

***

On the line https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L128
```
if (caller != owner) _approve(owner, msg.sender, allowance(owner, msg.sender) - shares);
```
the `caller` must be used instead of `msg.sender`.


