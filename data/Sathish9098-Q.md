# LOW FINDINGS

##

# NON CRITICAL FINDINGS 

##

## [NC -1]  USE A MORE RECENT VERSION OF SOLIDITY . LATEST SOLIDITY VERSION IS 0.8.17 . LATEST VERSION HAVE MORE ADVANCED FEATURES .

File : src/vault/PermissionRegistry.sol

     4 : pragma solidity ^0.8.15;

[Link To Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol)

File : src/vault/CloneRegistry.sol

    4 : pragma solidity ^0.8.15;

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol)

File : src/vault/DeploymentController.sol

     4 : pragma solidity ^0.8.15;

[Link To Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol)

File : src/vault/adapter/yearn/YearnAdapter.sol

     4 : pragma solidity ^0.8.15;

[Link To Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol)




##

## [NC-2] TYPOS

File : src/vault/CloneRegistry.sol

     /// @audit registerd   = >  registered
 
     14 :   * Is used by `VaultController` to check if a target is a registerd clone.

 

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol)








NC-1	Missing checks for address(0) when assigning values to address state variables	1
NC-2	Return values of approve() not checked	12
NC-3	Event is missing indexed fields	24
NC-4	Functions not used internally could be marked external	15

L-1	abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()	1
L-2	Do not use deprecated library functions	1
L-3	Empty Function Body - Consider commenting why	10
L-4	Initializers could be front-run	14
L-5	Unsafe ERC20 operation(s)	11

