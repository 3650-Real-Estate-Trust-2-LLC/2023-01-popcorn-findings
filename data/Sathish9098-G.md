##

###  [GAS-1] ++I/I++ OR --I/I-- SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} OR  UNCHECKED{--I}/UNCHECKED{I--} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop

> INSTANCES (20)

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

> INSTANCES (5)

File : src/vault/adapter/yearn/YearnAdapter.sol

      89 : function _shareValue(uint256 yShares) internal view returns (uint256) {

     101 :  function _freeFunds() internal view returns (uint256) {

     109 :  function _yTotalAssets() internal view returns (uint256) {

     114 :  function _calculateLockedProfit() internal view returns (uint256) {

[Link To Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol)

File : src/vault/VaultController.sol

    692:  function _verifyAdapterConfiguration(address adapter, bytes32 adapterId) internal view {


[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol)


##

###  [GAS-3]  CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS. 


Make it outside and only use it inside.

> INSTANCES (3)

File : src/utils/MultiRewardEscrow.sol

[L155-L167](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L155-L167)


[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol)

File : src/utils/MultiRewardStaking.sol

[MultiRewardStaking.sol#L171-L187](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L171-L187)

 [MultiRewardStaking.sol#L373-L382](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L373-L382)
 
[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol)

##

### [GAS-4] BYTES CONSTANTS ARE MORE EFFICIENT THAN STRING CONSTANTS

> INSTANCES (4)

File: src/vault/adapter/beefy/BeefyAdapter.sol

    24:   string internal _name;

    25:   string internal _symbol;

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol)

File : src/utils/MultiRewardStaking.sol

    31:   string private _name;

    32:   string private _symbol;

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol)

##

###  [GAS-5] <X> += <Y> OR <X> -= <Y> COSTS MORE GAS THAN <X> = <X> + <Y> OR <X> = <X> - <Y> FOR STATE VARIABLES . FOR EVERY CALL CAN SAVE 13 GAS 

> INSTANCES (4)

File : src/utils/MultiRewardStaking.sol


     408 :  rewardInfos[_rewardToken].index += deltaIndex;

     431:   accruedRewards[_user][_rewardToken] += supplierDelta;

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol)

File: src/vault/adapter/abstracts/AdapterBase.sol

     158 :  underlyingBalance += _underlyingBalance() - underlyingBalance_;

      225:   underlyingBalance -= underlyingBalance_ - _underlyingBalance();

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol)

##

### [GAS-6] Use uint256 instead of uint8 to save gas 

> INSTANCES (13)

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

###  [GAS-7]  USE FUNCTION INSTEAD OF MODIFIERS 

> INSTANCES (3)


File : src/vault/VaultController.sol

    modifier canCreate() {
    if (
      permissionRegistry.endorsed(address(1))
        ? !permissionRegistry.endorsed(msg.sender)
        : permissionRegistry.rejected(msg.sender)
     ) revert NotAllowed(msg.sender);
     _;
    }

[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol)

File : src/vault/Vault.sol

     modifier syncFeeCheckpoint() {
        _;
        highWaterMark = convertToAssets(1e18);
      }
##
      modifier takeFees() {
        uint256 managementFee = accruedManagementFee();
        uint256 totalFee = managementFee + accruedPerformanceFee();
        uint256 currentAssets = totalAssets();
        uint256 shareValue = convertToAssets(1e18);

        if (shareValue > highWaterMark) highWaterMark = shareValue;

        if (managementFee > 0) feesUpdatedAt = block.timestamp;

        if (totalFee > 0 && currentAssets > 0)
            _mint(feeRecipient, convertToShares(totalFee));

        _;
       }


[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)

##

### [GAS-8] Instead of using operator && on single IF condition . Using double IF checks possible to save more gas

  
File : src/vault/Vault.sol

    490 :  if (totalFee > 0 && currentAssets > 0)
   
[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)

##

### [GAS-9]  OPTIMIZE NAMES TO SAVE GAS

public/external function names and public member variable names can be optimized to save gas. See [this]()link for an example of how it works. In this report are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, per sorted position shifted.

> INSTANCES (15) 



