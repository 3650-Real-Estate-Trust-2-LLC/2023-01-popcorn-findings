## QA Report

| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | USE OF FLOATING PRAGMA | 35 |
| [L-2](#L-2) | RETURN VALUE OF LOW-LEVEL CALLS NOT CHECKED | 1 |
| [L-3](#L-3) | MISSING EXISTANCE CHECK IN `toggleTemplateEndorsement` METHOD | 1 |
| [L-4](#L-4) | `super` KEYWORD NOT USED TO CALL PARENT FUNCTIONS | 4 |
| [NC-1](#NC-1) | FUNCTION STATE MUTABILITY CAN BE RESTRICTED TO VIEW | 4 |
| [NC-2](#NC-2) | AVOID VARIABLE NAMES THAT CAN SHADE | 10 |
| [NC-3](#NC-3) | UNUSED LOCAL VARIABLE | 1 |
| [NC-4](#NC-4) | UNUSED FUNCTION PARAMETER | 1 |
| [NC-5](#NC-5) | ERROR DECLARED BUT NOT USED | 1 |
| [NC-6](#NC-6) | MISSING CHECKS FOR `address(0)` OUTSIDE OF AUTOMATED FINDINGS | 1 |
| [NC-7](#NC-7) | INTERFACE CODE DEVELOPED BUT NOT IMPLEMENTED | 1 |

### [L-1] USE OF FLOATING PRAGMA

Impact: [swcregistry](https://swcregistry.io/docs/SWC-103)

*Instances (35):*

All File in Scope uses `pragma solidity ^0.8.15;` Floating pragma Solidity Version.

### [L-2] RETURN VALUE OF LOW-LEVEL CALLS NOT CHECKED

*Instance (1):*
```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

444:     address(strategy).delegatecall(
445:          abi.encodeWithSignature("harvest()")
446:     );

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L444-L446)

### [L-3] MISSING EXISTANCE CHECK IN `toggleTemplateEndorsement` METHOD

In `toggleTemplateEndorsement` method of `TemplateRegistry` contract, there is only one check `!templateExists[templateId]` which checks if the templateId exists or not.

But it is not necessary that the given templateId is mapped to templateCategory which was passed through function parameter. So it is important to check this by adding:

```
    if(templates[templateCategory][templateId].implementation == address(0)) revert CategoryIdMismatch(templateCategory, templateId);
```

### [L-4] `super` KEYWORD NOT USED TO CALL PARENT FUNCTIONS

It is recommended to use the `super` keyword to call a function in parent contract.

*Instances (4):*
```solidity
File: src/utils/MultiRewardStaking.sol

76:     return deposit(_amount, msg.sender);

80:     return mint(_amount, msg.sender);

84:     return withdraw(_amount, msg.sender, msg.sender);

88:     return redeem(_amount, msg.sender, msg.sender);

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol)

### [NC-1] FUNCTION STATE MUTABILITY CAN BE RESTRICTED TO VIEW

Following function instances do not change state of the contract. So it is recommended to restrict its mutability to view only.

*Instances (4):*
```solidity
File: src/utils/MultiRewardStaking.sol

351:     function _calcRewardsEnd(

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L351)

```solidity
File: src/vault/VaultController.sol

242:     function _encodeAdapterData(DeploymentArgs memory adapterData, bytes memory baseAdapterData)

667:     function _verifyCreatorOrOwner(address vault) internal returns (VaultMetadata memory metadata) {

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol)

```solidity
File: src/vault/strategy/StrategyBase.sol

12:    function verifyAdapterSelectorCompatibility(bytes4[8] memory sigs) public {

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/strategy/StrategyBase.sol#L12)

### [NC-2] AVOID VARIABLE NAMES THAT CAN SHADE

`_totalAssets` shades with `_totalAssets` function. Other shades are self explanatory.

*Instance (10):*
```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

388:     uint256 _totalAssets = totalAssets();

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L388)

```solidity
File: src/utils/MultiRewardEscrow.sol

117:      token: token,
118:      start: start,
123:      account: account

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L117-L123)

```solidity
File: src/utils/MultiRewardStaking.sol

267:     escrowPercentage: escrowPercentage,
268:     escrowDuration: escrowDuration,
269:     offset: offset

280:     ONE: ONE,
281:     rewardsPerSecond: rewardsPerSecond,
282:     rewardsEndTimestamp: rewardsEndTimestamp,

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L351)


### [NC-3] UNUSED LOCAL VARIABLE

Local variable should be removed if not used. Either use this variable in place of state variable to save gas or remove this variable.

*Instance (1)*
```solidity
File: src/vault/Vault.sol

448:     uint256 highWaterMark_ = highWaterMark;

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#448)

### [NC-4] UNUSED FUNCTION PARAMETER

Unused parameter can be removed.

*Instance (1):*
```solidity
File: src/vault/VaultController.sol

390:     function _registerVault(address vault, VaultMetadata memory metadata) internal {

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#390)

### [NC-5] ERROR DECLARED BUT NOT USED

Error should be removed as it is not used.

*Instance (1):*
```solidity
File: src/vault/CloneFactory.sol

32:     error NotEndorsed(bytes32 templateKey);

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneFactory.sol#32)

### [NC-6] MISSING CHECKS FOR `address(0)` OUTSIDE OF AUTOMATED FINDINGS

*Instance (1):*
```solidity
File: src/vault/CloneRegistry.sol

44:       address clone

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneFactory.sol#44)

### [NC-7] INTERFACE CODE DEVELOPED BUT NOT IMPLEMENTED

The contracts listed below implements all the methods of the interface `IPermissionRegistry`, `IVaultRegistry`, `IMultiRewardEscrow` and `IMultiRewardStaking` respectively. But those interfaces are not added in `is`.

```solidity
File: src/vault/PermissionRegistry.sol

14:       contract PermissionRegistry is Owned {

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#14)

```solidity
File: src/vault/VaultRegistry.sol

15:       contract VaultRegistry is Owned {

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#15)

```solidity
File: src/utils/MultiRewardEscrow.sol

21:       contract MultiRewardEscrow is Owned {

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#21)

```solidity
File: src/utils/MultiRewardStaking.sol

26:       contract MultiRewardStaking is ERC4626Upgradeable, OwnedUpgradeable {

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#26)