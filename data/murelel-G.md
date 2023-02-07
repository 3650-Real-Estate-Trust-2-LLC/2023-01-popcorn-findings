# Gas Optimizations


# [GAS-01] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

#### Proof Of Concept
```solidity
53: for (uint256 i = 0; i < escrowIds.length; i++) {

155: for (uint256 i = 0; i < escrowIds.length; i++) {

210: for (uint256 i = 0; i < tokens.length; i++) {


```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol

```solidity
171: for (uint8 i; i < _rewardTokens.length; i++) {

373: for (uint8 i; i < _rewardTokens.length; i++) {

```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L461

```
42: for (uint256 i = 0; i < len; i++) {
 ```
 
 https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L42
 
 ```solidity
319: for (uint8 i = 0; i < len; i++) {
 
337: for (uint8 i = 0; i < len; i++) {

357: for (uint8 i = 0; i < len; i++) {

374: for (uint8 i = 0; i < len; i++) {

437: for (uint256 i = 0; i < len; i++) {

494: for (uint256 i = 0; i < len; i++) {

523: for (uint256 i = 0; i < len; i++) {

564: for (uint256 i = 0; i < len; i++) {

587: for (uint256 i = 0; i < len; i++) {

607: for (uint256 i = 0; i < len; i++) {

620: for (uint256 i = 0; i < len; i++) {

633: for (uint256 i = 0; i < len; i++) {

646: for (uint256 i = 0; i < len; i++) {

766: for (uint256 i = 0; i < len; i++) {

806: for (uint256 i = 0; i < len; i++) {

 
 ```
 
 https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol
 


# [GAS-02] <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables

```solidity 
408: rewardInfos[_rewardToken].index += deltaIndex;

431: accruedRewards[_user][_rewardToken] += supplierDelta;
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol


#  [GAS-03] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

```
19: function execute(address target, bytes calldata callData) external onlyOwner returns (bool success, bytes memory returndata)
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#LL19C3-L22C52

```solidity
500: function setHarvestCooldown(uint256 newCooldown) external onlyOwne
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L500

```solidity 
549: function setPerformanceFee(uint256 newFee) public onlyOwner {
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L549

```
574: function pause() external onlyOwner {
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L574

```
582: function unpause() external onlyOwner {
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L582

```
41: function addClone(
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L41

# [GAS-04] Using bools for storage incurs overhead

// Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.

```
struct Permission {
  bool endorsed;
  bool rejected;
}
````
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IPermissionRegistry.sol#LL8-L11C2

```
struct Template {
  /// @Notice Cloneable implementation address
  address implementation;
  /// @Notice implementations can only be cloned if endorsed
  bool endorsed;
  /// @Notice Optional - Metadata CID which can be used by the frontend to add informations to a vault/adapter...
  string metadataCid;
  /// @Notice If true, the implementation will require an init data to be passed to the clone function
  bool requiresInitData;
  /// @Notice Optional - Address of an registry which can be used in an adapter initialization
  address registry;
  /// @Notice Optional - Only used by Strategies. EIP-165 Signatures of an adapter required by a strategy
  bytes4[8] requiredSigs;
}
````
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ITemplateRegistry.sol#LL8-L21C2

# [GAS-5] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

```
33: uint8 private _decimals;
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#LL33C3-L33C27

```
171: for (uint8 i; i < _rewardTokens.length; i++) {
373: for (uint8 i; i < _rewardTokens.length; i++) {
450: uint8 v,
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol

```
38:  uint8 internal _decimals;
637: uint8 v,

```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol

```solidity
38: uint8 internal _decimals;
669:  uint8 v,
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol

```solidity
314: uint8 len = uint8(vaults.length);
319:    for (uint8 i = 0; i < len; i++) {
336:    uint8 len = uint8(vaults.length);
337:    for (uint8 i = 0; i < len; i++) {
353:    uint8 len = uint8(vaults.length);
357:    for (uint8 i = 0; i < len; i++) {
373:    uint8 len = uint8(vaults.length);
374:    for (uint8 i = 0; i < len; i++) {
436:    uint8 len = uint8(vaults.length);
488:    uint8 len = uint8(vaults.length);
517:    uint8 len = uint8(vaults.length);
563:    uint8 len = uint8(templateCategories.length);
583:    uint8 len = uint8(templateCategories.length);
606:    uint8 len = uint8(vaults.length);
619:    uint8 len = uint8(vaults.length);
632:    uint8 len = uint8(vaults.length);
645:    uint8 len = uint8(vaults.length);
765:    uint8 len = uint8(adapters.length);
805:    uint8 len = uint8(adapters.length);
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol