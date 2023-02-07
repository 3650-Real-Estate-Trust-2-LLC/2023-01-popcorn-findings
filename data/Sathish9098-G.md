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

File : src/utils/MultiRewardStaking.sol

   171:  for (uint8 i; i < _rewardTokens.length; i++) {

   373:  for (uint8 i; i < _rewardTokens.length; i++) {
 
[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol)

File : src/vault/VaultController.sol

    319:  for (uint8 i = 0; i < len; i++) {

   337:   for (uint8 i = 0; i < len; i++) {

  357:   for (uint8 i = 0; i < len; i++) {

  374:    for (uint8 i = 0; i < len; i++) {

  437:    for (uint256 i = 0; i < len; i++) {

  494:   for (uint256 i = 0; i < len; i++) {

  523:   for (uint256 i = 0; i < len; i++) {

  564:  for (uint256 i = 0; i < len; i++) {

  587:  for (uint256 i = 0; i < len; i++) {

  607:  for (uint256 i = 0; i < len; i++) {

   620:  for (uint256 i = 0; i < len; i++) {

   633:  for (uint256 i = 0; i < len; i++) {

   646:  for (uint256 i = 0; i < len; i++) {

   766:  for (uint256 i = 0; i < len; i++) {

   806:  for (uint256 i = 0; i < len; i++) {

[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol)

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

File : src/utils/MultiRewardStaking.sol

   [MultiRewardStaking.sol#L171-L187](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L171-L187)

  [MultiRewardStaking.sol#L373-L382](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L373-L382)
 
[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol)

##

### [GAS-4] BYTES CONSTANTS ARE MORE EFFICIENT THAN STRING CONSTANTS

File: src/vault/adapter/beefy/BeefyAdapter.sol

    24:   string internal _name;

    25:   string internal _symbol;

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol)

File : src/utils/MultiRewardStaking.sol

  31:   string private _name;

  32:   string private _symbol;

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol)

##

## [GAS-5] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES . FOR EVERY CALL CAN SAVE 13 GAS 


File : src/utils/MultiRewardStaking.sol


     408 :  rewardInfos[_rewardToken].index += deltaIndex;

    431:   accruedRewards[_user][_rewardToken] += supplierDelta;

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol)

File: src/vault/adapter/abstracts/AdapterBase.sol

   158 :  underlyingBalance += _underlyingBalance() - underlyingBalance_;

   225:   underlyingBalance -= underlyingBalance_ - _underlyingBalance();

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol)

##

## [GAS-6] Use uint256 instead of uint8 to save gas 

File : src/vault/VaultController.sol


   436 :   uint8 len = uint8(vaults.length);

  488:     uint8 len = uint8(vaults.length);

  517:    uint8 len = uint8(vaults.length);

  563:   uint8 len = uint8(templateCategories.length);

  583:   uint8 len = uint8(templateCategories.length);

  606:   uint8 len = uint8(vaults.length);

  619:   uint8 len = uint8(vaults.length);

632:    uint8 len = uint8(vaults.length);

  645:  uint8 len = uint8(vaults.length);

 765:  uint8 len = uint8(adapters.length);

 805:  uint8 len = uint8(adapters.length);

[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol)

File: src/vault/adapter/abstracts/AdapterBase.sol

   38:  uint8 internal _decimals;

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol)

##


  

   






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