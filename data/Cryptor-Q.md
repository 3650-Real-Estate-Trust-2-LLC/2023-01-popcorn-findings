##Claim rewards has no access control 
Claim rewards function has no access control which can lead to users receiving funds earlier than they want

##ChangeVaultFees has not restriction on the amount of times it can be called 
The function ChangeVaultfees has 0 restriction on the amount of time that it can be called. A vault controller can call Changevaultfees with a quit period of 1 day more than once to change the fee. The first one can seem reasonable and the subsequent call will be more malicious.


##Constructor in Multi-reward staking has no 0 address checks 
the constructor in multirewardstaking.sol does not a have a check for 0 addresses

##Vault Controller can front run a user by calling pause() as a grieving attack
A vaultcontroller that doesn't want a user to make deposit in his/her vault can call pause() with a higher gas fee

## uint256 highWaterMark_ in  function accruedPerformanceFee is unused 

## function takeManagementAndPerformanceFees is empty 

## Avoid having functions of the same function 
functions should not have the same name 

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L124

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L134