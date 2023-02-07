1. 
outdated solidity version
current version: 0.8.15

minimum required version must be `0.8.18`
new features:
```
Support for Paris Hardfork
Deprecation of selfdestruct
more new features: https://blog.soliditylang.org/2023/02/01/solidity-0.8.18-release-announcement/
```

Recommended Mitigation Steps
change `pragma solidity ^0.8.15;` to `pragma solidity ^0.8.18;`


2. 
missing `__ReentrancyGuard_init()` in vault initialize function
`Vault.sol#L57-L98`

Recommended Mitigation Steps
add missing init into initialize function


3. 
`Vault.sol#54 :: initialize` can be called by anybody when contract is not initialized
technically its impossible to call it when clone factory call it in the same transaction 
as creating the Vault but its nice to have a check for msg.sender to be `cloneFactory` 



4. 
miscalculation for deposit amount
`Vault.sol#L143` 
because of rouding down in `feeShares` calcualtion for deposit amount 
feeRecipient could lost fee if deposited amount is small or under a certain amount

Recommended Mitigation Steps
using rounding up 
or only using rounding up when last calculated fee is 0


5. 
badUser could emit infinite number of Events
`Vault.sol#L256-L282 :: redeem` & `Vault.sol#L215-L244 :: withdraw`
badUser could emit infinite number of Withdraw Events beacuse (`redeem` OR `withdraw`) do not revert if 
`shares` is 0 also users transactions will be go through but nothing will happens 

`receiver` & `owner` params can be any user address attacker choose

Recommended Mitigation Steps
adding a check for 0 amount of assets or shares
e.g.:
```
if(shares == 0) revert ZeroAmount();
// OR
if(assets == 0) revert ZeroAmount();
```



6. 
NatSpec comments should be much more 

multiple functions do not have `@param` or `@return` tags in their comments  
also couple of functions dont even have comments 

for example `MutiRewardStaking.sol#L191-L202 :: _lockToken` wich do not have `param` tags 
should be like this:
```
/**
	* @notice Locks a percentage of a reward in an escrow contract. Pays out the rest to the user.
	* @param user address of reward receiver 
	* @param rewardToken address of ERC20 reward token
	* @param rewardAmount amount of reward that user receive 
*/
```


7.
`VaultController.sol#L242`, `VaultController.sol#L667`: Function visibilty can be restricted to view
`Vault.sol#L447`: cached local variable did not used instead function sload state variable highWaterMark three time
