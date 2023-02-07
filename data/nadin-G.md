## Gas Optimizations
Issue  
## [G-01] SETTING THE CONSTRUCTOR TO PAYABLE
## [G-02] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS
## [G-03] STORAGE POINTER TO A STRUCTURE IS CHEAPER THAN COPYING EACH VALUE OF THE STRUCTURE INTO MEMORY, SAME FOR ARRAY AND MAPPING
## [G-04] nonce can be uint96 to optimize storage read/writes
## [G-05] Increments can be unchecked (and made postfix)
## [G-06]  Using unchecked blocks to save gas
## [G-07] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES
## [G‑08] KECCAK256() SHOULD ONLY NEED TO BE CALLED ON A SPECIFIC STRING LITERAL ONCE
## [G‑09] AVOID CONTRACT EXISTENCE CHECKS BY USING LOW LEVEL CALLS
## [G-10] The solady Library's Ownable contract is significantly gas-optimized, which can be used
## [G-11] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS
Total: 97 contexts over 11 issues

## [G-01] SETTING THE CONSTRUCTOR TO PAYABLE
Saves ~13 gas per instance
Context: 08
File: src/vault/PermissionRegistry.sol

    20:  constructor(address _owner) Owned(_owner) {}

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L20

File: src/vault/AdminProxy.sol

    16:  constructor(address _owner) Owned(_owner) {}

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/AdminProxy.sol#L16

File: src/vault/CloneRegistry.sol

    22:  constructor(address _owner) Owned(_owner) {}

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneRegistry.sol#L22

File: src/vault/VaultRegistry.sol

    21:  constructor(address _owner) Owned(_owner) {}

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L21

File: src/vault/DeploymentController.sol

    35:  constructor(

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/DeploymentController.sol#L35-L44

File: src/vault/TemplateRegistry.sol

    24:   constructor(address _owner) Owned(_owner) {}

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L24

File: src/utils/MultiRewardEscrow.sol

    30:  constructor(address _owner, address _feeRecipient) Owned(_owner) {

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L30-L32

File: src/vault/VaultController.sol

    53:  constructor(

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L53-L70

## [G-02] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS:
The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop.
Context: 21
File: src/vault/PermissionRegistry.sol

    42:    for (uint256 i = 0; i < len; i++) {

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L42

File: src/utils/MultiRewardEscrow.sol

    53:    for (uint256 i = 0; i < escrowIds.length; i++) {
    155:    for (uint256 i = 0; i < escrowIds.length; i++) {
    210:    for (uint256 i = 0; i < tokens.length; i++) {

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L53
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L155
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L210

File: src/utils/MultiRewardStaking.sol 

    171:    for (uint8 i; i < _rewardTokens.length; i++) {
    373:    for (uint8 i; i < _rewardTokens.length; i++) {

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L171
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L373

File: src/vault/VaultController.sol

    319:    for (uint8 i = 0; i < len; i++) {
    337:    for (uint8 i = 0; i < len; i++) {
    357:    for (uint8 i = 0; i < len; i++) {
    374:    for (uint8 i = 0; i < len; i++) {
    437:    for (uint256 i = 0; i < len; i++) {
    494:    for (uint256 i = 0; i < len; i++) {
    523:    for (uint256 i = 0; i < len; i++) {
    564:    for (uint256 i = 0; i < len; i++) {
    587:    for (uint256 i = 0; i < len; i++) {
    607:    for (uint256 i = 0; i < len; i++) {
    620:    for (uint256 i = 0; i < len; i++) {
    633:    for (uint256 i = 0; i < len; i++) {
    646:    for (uint256 i = 0; i < len; i++) {
    766:    for (uint256 i = 0; i < len; i++) {
    806:    for (uint256 i = 0; i < len; i++) {

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L319
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L337
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L357
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L374
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L437
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L494
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L523
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L564
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L587
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L607
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L620
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L633
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L646
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L766
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L806

## [G-03] STORAGE POINTER TO A STRUCTURE IS CHEAPER THAN COPYING EACH VALUE OF THE STRUCTURE INTO MEMORY, SAME FOR ARRAY AND MAPPING
It may not be obvious, but every time you copy a storage struct/array/mapping to a memory variable, you are literally copying each member by reading it from storage, which is expensive. And when you use the storage keyword, you are just storing a pointer to the storage, which is much cheaper.
Context: 12
File: src/vault/DeploymentController.sol

    104:    Template memory template = templateRegistry.getTemplate(templateCategory, templateId);

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/DeploymentController.sol#L104

File: src/utils/MultiRewardEscrow.sol

    52:    Escrow[] memory selectedEscrows = new Escrow[](escrowIds.length);
    157:      Escrow memory escrow = escrows[escrowId];

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L52
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L157

File: src/utils/MultiRewardStaking.sol 

    176:      EscrowInfo memory escrowInfo = escrowInfos[_rewardTokens[i]];
    254:    RewardInfo memory rewards = rewardInfos[rewardToken];
    297:    RewardInfo memory rewards = rewardInfos[rewardToken];
    328:    RewardInfo memory rewards = rewardInfos[rewardToken];
    372:    IERC20[] memory _rewardTokens = rewardTokens;
    375:      RewardInfo memory rewards = rewardInfos[rewardToken];
    414:    RewardInfo memory rewards = rewardInfos[_rewardToken];

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L176
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L254
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L297
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L328
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L372
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L375
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L414

File: src/vault/VaultController.sol

    155:    address[] memory vaultContracts = new address[](1);
    156:    bytes[] memory rewardsDatas = new bytes[](1);

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L155-L156

## [G-04] nonce can be uint96 to optimize storage read/writes
Context: 01
File: src/utils/MultiRewardEscrow.sol

    71:  uint256 internal nonce;

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L71

## [G-05] Increments can be unchecked (and made postfix)
Context: 01
File: src/utils/MultiRewardEscrow.sol

    102:    nonce++;

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L102

## [G-06]  Using unchecked blocks to save gas
Context: 07
File: src/utils/MultiRewardEscrow.sol

    110:      amount -= fee;
    162:      escrows[escrowId].balance -= claimable;

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L110
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L162

File: src/vault/adapter/beefy/BeefyAdapter.sol

    178:            assets -= assets.mulDiv(

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L178

File: src/utils/MultiRewardStaking.sol 

    198:    uint256 payout = rewardAmount - escrowed;
    393:      elapsed = block.timestamp - rewards.lastUpdatedTimestamp;
    395:      elapsed = rewards.rewardsEndTimestamp - rewards.lastUpdatedTimestamp;
    424:    uint256 deltaIndex = rewards.index - oldIndex;

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L198
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L393
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L395
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L424

## [G-07] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES
Context: 12
File: src/utils/MultiRewardEscrow.sol

    110:      amount -= fee;
    162:      escrows[escrowId].balance -= claimable;

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L110
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L162

File: src/vault/adapter/beefy/BeefyAdapter.sol

    178:            assets -= assets.mulDiv(

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L178

File: src/utils/MultiRewardStaking.sol 

    357:      amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);
    408:    rewardInfos[_rewardToken].index += deltaIndex;
    431:    accruedRewards[_user][_rewardToken] += supplierDelta;

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L357
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L408
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L431

File: src/vault/Vault.sol

    228:        shares += feeShares;
    343:        shares += shares.mulDiv(
    365:        assets += assets.mulDiv(
    387:        assets -= assets.mulDiv(

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L228
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L343
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L365
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L387

File: src/vault/adapter/abstracts/AdapterBase.sol

    158:        underlyingBalance += _underlyingBalance() - underlyingBalance_;
    225:            underlyingBalance -= underlyingBalance_ - _underlyingBalance();

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L158
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L225

## [G‑08] KECCAK256() SHOULD ONLY NEED TO BE CALLED ON A SPECIFIC STRING LITERAL ONCE
It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once.
Context: 06
File: src/utils/MultiRewardStaking.sol 

    466:                keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),
    495:          keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L466
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L495

File: src/vault/Vault.sol 

    685:                                keccak256(
    686:                                    "Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"
    720:                    keccak256(
    721:                        "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L685-L686
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L720-L721

File: src/vault/adapter/abstracts/AdapterBase.sol

    653:                                keccak256(
    654:                                    "Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"
    688:                    keccak256(
    689:                        "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L653-L654
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L688-L689

## [G‑09] AVOID CONTRACT EXISTENCE CHECKS BY USING LOW LEVEL CALLS
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.
Context: 22
File: src/vault/VaultController.sol

    126:    (bool success, bytes memory returnData) = adminProxy.execute(
    230:    (bool success, bytes memory returnData) = adminProxy.execute(
    260:    (bool success, bytes memory returnData) = adminProxy.execute(
    288:    (bool success, bytes memory returnData) = adminProxy.execute(
    323:      (bool success, bytes memory returnData) = adminProxy.execute(
    338:      (bool success, bytes memory returnData) = adminProxy.execute(
    360:      (bool success, bytes memory returnData) = adminProxy.execute(
    375:      (bool success, bytes memory returnData) = adminProxy.execute(
    391:    (bool success, bytes memory returnData) = adminProxy.execute(
    410:    (bool success, bytes memory returnData) = adminProxy.execute(
    450:      (bool success, bytes memory returnData) = adminProxy.execute(
    459:      (success, returnData) = adminProxy.execute(
    497:      (bool success, bytes memory returnData) = adminProxy.execute(
    545:    (bool success, bytes memory returnData) = adminProxy.execute(
    565:      (bool success, bytes memory returnData) = adminProxy.execute(
    588:      (bool success, bytes memory returnData) = adminProxy.execute(
    609:      (bool success, bytes memory returnData) = adminProxy.execute(
    622:      (bool success, bytes memory returnData) = adminProxy.execute(
    635:      (bool success, bytes memory returnData) = adminProxy.execute(
    648:      (bool success, bytes memory returnData) = adminProxy.execute(
    767:      (bool success, bytes memory returnData) = adminProxy.execute(
    807:      (bool success, bytes memory returnData) = adminProxy.execute(

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L126
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L230
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L260
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L288
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L323
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L338
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L360
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L375
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L391
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L410
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L450
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L459
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L497
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L545
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L565
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L588
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L609
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L622
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L635
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L648
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L767
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L807

## [G-10] The solady Library's Ownable contract is significantly gas-optimized, which can be used
The project uses the onlyOwner authorization model I recommend using Solady's highly gas optimized contract.

https://github.com/Vectorized/solady/blob/main/src/auth/OwnableRoles.sol

## [G-11] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS
It is not necessary to have both a named return and a return statement.
Context: 07
File: src/utils/MultiRewardEscrow.sol 

    51:  function getEscrows(bytes32[] calldata escrowIds) external view returns (Escrow[] memory) {

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L51

File: src/vault/Vault.sol

    326:        returns (uint256 shares)
    340:    function previewMint(uint256 shares) public view returns (uint256 assets) {
    361:        returns (uint256 shares)
    383:        returns (uint256 assets)

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L324-L326
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L340
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L358-L361
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L380-L383

File: src/vault/VaultController.sol

    673:  function _verifyCreator(address vault) internal view returns (VaultMetadata memory metadata) {

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L673

File: src/vault/adapter/abstracts/AdapterBase.sol

    385:        returns (uint256 shares)

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L380-L385

