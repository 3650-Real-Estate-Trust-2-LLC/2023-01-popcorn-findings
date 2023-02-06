##

###  [GAS-1] ++I/I++ OR --I/I-- SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} OR  UNCHECKED{--I}/UNCHECKED{I--} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop

File : src/vault/PermissionRegistry.sol

    42:  for (uint256 i = 0; i < len; i++) {

[Link To Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol)

File : src/utils/MultiRewardEscrow.sol

    53 :  for (uint256 i = 0; i < escrowIds.length; i++) {

   155 :  for (uint256 i = 0; i < escrowIds.length; i++) {

   210 :  for (uint256 i = 0; i < tokens.length; i++) {

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol)

##

###  [GAS-2]  INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

File : src/vault/adapter/yearn/YearnAdapter.sol

     89 : function _shareValue(uint256 yShares) internal view returns (uint256) {

    101 :  function _freeFunds() internal view returns (uint256) {

    109 :  function _yTotalAssets() internal view returns (uint256) {

    114 :  function _calculateLockedProfit() internal view returns (uint256) {

[Link To Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol)

##

###  [GAS-3]  CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS. 


Make it outside and only use it inside.

File : src/utils/MultiRewardEscrow.sol

[L155-L167](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L155-L167)


[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol)



  

   






GAS-1	Use assembly to check for address(0)	24
GAS-2	Using bools for storage incurs overhead	3
GAS-3	Cache array length outside of loop	5
GAS-4	State variables should be cached in stack variables rather than re-reading them from storage	3
GAS-5	Use calldata instead of memory for function arguments that do not get mutated	16
GAS-6	Don't initialize variables with default value	19
GAS-7	Functions guaranteed to revert when called by normal users can be marked payable	28
GAS-8	++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too)	24
GAS-9	Using private rather than public for constants, saves gas	1
GAS-10	Use != 0 instead of > 0 for unsigned integer comparison	23