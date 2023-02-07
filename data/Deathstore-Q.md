### Unable to add token's with more than 20 decimals
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L274
` uint64 ONE = (10**IERC20Metadata(address(rewardToken)).decimals()).safeCastTo64();`
If owner try to add token's such as NEAR 
https://etherscan.io/token/0x85f17cf997934a597031b2e18a9ab6ebd4b9f6a4
(used for bridge)
transaction will be reverted on safecast64, because 10**20>2^64

### Extra boolean initialize 
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneFactory.sol#L42
There is no any visible point to initialize success as true, but it might be use in some malicious action's

### Not using safecast
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L314
In this line and on another where explicit conversation used it better to use safecast. (in this line safecast8()). User can send to this 257 vault's and 1 new adapter. He put more gas in it, there is no any exlpoit. But it seems better to revert transation's like this.