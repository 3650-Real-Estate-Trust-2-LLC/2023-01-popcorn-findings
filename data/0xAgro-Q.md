# QA Report
## Finding Summary

[**Low Severity**](#Low-Severity)
1. [**Relatively Small block.timestamp Cast**](#1-Relatively-Small-blocktimestamp-Cast)
2. [**Unchecked Cast May Overflow**](#2-Unchecked-Cast-May-Overflow)
3. [**TODO Left in Production Code**](#3-TODO-Left-in-Production-Code)*

[**Non-Critical**](#Non-Critical)
1. [**Use of Exponentiation Over Scientific Notation**](#1-Use-of-Exponentiation-Over-Scientific-Notation)
2. [**Use of ecrecover**](#2-Use-of-ecrecover)
3. [**Order of Functions Not Compliant With Solidity Docs**](#3-Order-of-Functions-Not-Compliant-With-Solidity-Docs)
4. [**Contract Layout Voids Solidity Docs**](#4-Contract-Layout-Voids-Solidity-Docs)
5. [**Long Lines**](#5-Long-Lines)
6. [**Order of Modifiers Not Compliant With Solidity Docs**](#6-Order-of-Modifiers-Not-Compliant-With-Solidity-Docs)
7. [**Contracts Missing @title Tag**](#7-Contracts-Missing-title-Tag)
8. [**Inconsistent Comment Initial Spacing**](#8-Inconsistent-Comment-Initial-Spacing)
9. [**Inconsistent Named Returns**](#9-Inconsistent-Named-Returns)
10. [**Spelling Mistakes**](#10-Spelling-Mistakes)
11. [**Inconsistent Header Centering**](#11-Inconsistent-Header-Centering)
12. [**Low Coverage**](#12-Low-Coverage)
13. [**Inconsistent Pragma Spacing**](#13-Inconsistent-Pragma-Spacing)
14. [**bytes.concat() Can Be Used Over abi.encodePacked()**](#14-bytesconcat-can-be-used-over-abiencodepacked)
15. [**Inconsistent Comment Capitalization**](#15-Inconsistent-Comment-Capitalization)
16. [**Named Returns Not Returning Named Variable**](#16-Named-Returns-Not-Returning-Named-Variable)
17. [**Inconsistent Trailing Period**](#17-Inconsistent-Trailing-Period)
18. [**Confusing Word shortening**](#18-Confusing-Word-shortening)
19. [**Inconsistent Headers**](#19-Inconsistent-Headers)
20. [**Lack of Overflow Error Message**](#20-Lack-of-Overflow-Error-Message)
21. [**Unused Cache Variable**](#21-Unused-Cache-Variable)
22. [**Add view Modifier**](#22-Add-view-Modifier)
23. [**Unused Param**](#23-Unused-Param)
24. [**Assumptive Language**](#24-Assumptive-Language)
25. [**Lack of Interface Documentation**](#25-Lack-of-interface-Documentation)

# Low Severity

## 1. Relatively Small block.timestamp Cast

As of now `block.timestamp.safeCastTo32()` will always revert when the `block.timestamp` is greater than `type(uint32).max` := 4294967295. [`block.timestamp` returns the current seconds since unix epoch](https://docs.soliditylang.org/en/v0.8.13/units-and-global-variables.html). A unix time of 4294967295 would imply a date and time of **Sunday, 7 February 2106 06:28:15 GMT** (can be verified [here](https://www.epochconverter.com/)). In approximetly 83 years all functions that implement the call `block.timestamp.safeCastTo32()` will always revert.

*/src/utils/MultiRewardEscrow.sol*

```solidity
114:	uint32 start = block.timestamp.safeCastTo32() + offset;
163:	escrows[escrowId].lastUpdateTime = block.timestamp.safeCastTo32();
175:	block.timestamp.safeCastTo32() < escrow.start
```

*/src/utils/MultiRewardStaking.sol*

```solidity
276:	? block.timestamp.safeCastTo32()
284:	lastUpdatedTimestamp: block.timestamp.safeCastTo32()
309:	prevEndTime > block.timestamp ? prevEndTime : block.timestamp.safeCastTo32(),
346:	rewardInfos[rewardToken].lastUpdatedTimestamp = block.timestamp.safeCastTo32();
409:	rewardInfos[_rewardToken].lastUpdatedTimestamp = block.timestamp.safeCastTo32();
```

## 2. Unchecked Cast May Overflow

As of Solidity 0.8 overflows are handled automatically; however, not for casting. For example `uint32(4294967300)` will result in `4` without reversion. Consider using a SafeCast.

*/src/vault/VaultController.sol*

```solidity
314:	uint8 len = uint8(vaults.length);
336:	uint8 len = uint8(vaults.length);
353:	uint8 len = uint8(vaults.length);
373:	uint8 len = uint8(vaults.length);
436:	uint8 len = uint8(vaults.length);
488:	uint8 len = uint8(vaults.length);
517:	uint8 len = uint8(vaults.length);
563:	uint8 len = uint8(templateCategories.length);
583:	uint8 len = uint8(templateCategories.length);
606:	uint8 len = uint8(vaults.length);
619:	uint8 len = uint8(vaults.length);
632:	uint8 len = uint8(vaults.length);
645:	uint8 len = uint8(vaults.length);
765:	uint8 len = uint8(adapters.length);
805:	uint8 len = uint8(adapters.length);
```

## 3. TODO Left in Production Code

*It is understood that the knowledge of this TODO and generally why TODOs should be removed is understood by the team [here](https://github.com/code-423n4/2023-01-popcorn#additional-context). This issue is left in as it does not technically void scope and can be beneficial for future auditors*.

`TODO`s should not be in production code. Each `TODO` should either be discarded or implemented if needed prior to production.

*/src/vault/adapter/abstracts/AdapterBase.sol*

```solidity
516:    // TODO use deterministic fee recipient proxy
```

# Non-Critical

## 1. Use of Exponentiation Over Scientific Notation

For better style and less computation consider replacing any power of 10 exponentiation (`10**3`) with its equivalent scientific notation (`1e3`).

*/src/vault/adapter/yearn/YearnAdapter.sol*

```solidity
25:	uint256 constant DEGRADATION_COEFFICIENT = 10**18;
```

## 2. Use of ecrecover

Consider using OpenZeppelin's ECDSA contract instead of raw `ecrecover`. 

*/src/utils/MultiRewardStaking.sol*

```solidity
459:	address recoveredAddress = ecrecover(
```

*/src/vault/Vault.sol*

```solidity
678:	address recoveredAddress = ecrecover(
```

## 3. Order of Functions Not Compliant With Solidity Docs

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) suggests the following function order: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private.

The following contracts are not compliant (examples are only to prove the functions are out of order NOT a full description): 

* [YearnAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol): public functions are positioned after internal functions.
* [MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol): external functions are positioned after internal functions.
* [BeefyAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol): public functions are positioned after internal functions.
* [MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol): external functions are positioned after public functions.
* [Vault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol): external functions are positioned after public functions.
* [VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol): external functions are positioned after internal functions.
* [AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol): public functions are positioned after internal functions.

## 4. Contract Layout Voids Solidity Docs

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout) suggests the following contract layout order: Type declarations, State variables, Events, Modifiers, Functions.

The following contracts are not compliant (examples are only to prove the layout are out of order NOT a full description): 

* [CloneFactory.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneFactory.sol): Events are positioned after Constructor (Functions).
* [PermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol): State is positioned after Constructor (Functions).
* [CloneRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol): State is positioned after Constructor (Functions).
* [VaultRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol): State is positioned after Constructor (Functions).
* [TemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol): State is positioned after Constructor (Functions).
* [MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol): State is positioned after Constructor (Functions).
* [MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol): Events are positioned after Functions.
* [Vault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol): Events are positioned after Functions.
* [VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol): Events are positioned after Constructor (Functions).
* [AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol): State is positioned after Functions

## 5. Long Lines

Lines with greater length than 120 characters are used. The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-lengthhttps://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) suggests that all lines should be 120 characters or less in width.

The following lines are longer than 120 characters, it is suggested to shorten these lines:

*/src/vault/AdminProxy.sol*
*  [13](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/AdminProxy.sol#L13). 

*/src/vault/VaultRegistry.sol*
*  [41](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol#L41). 

*/src/vault/DeploymentController.sol*
*  [91](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol#L91). 

*/src/vault/TemplateRegistry.sol*
*  [49](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol#L49). 

*/src/vault/adapter/yearn/YearnAdapter.sol*
*  [6](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L6). 

*/src/utils/MultiRewardEscrow.sol*
*  [6](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L6), [49](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L49). 

*/src/vault/adapter/beefy/BeefyAdapter.sol*
*  [6](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L6), [16](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L16). 

*/src/utils/MultiRewardStaking.sol*
*  [6](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L6), [7](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L7), [22](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L22), [106](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L106), [120](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L120), [291](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L291), [380](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L380). 

*/src/vault/Vault.sol*
*  [6](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L6), [55](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L55), [398](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L398), [408](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L408), [425](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L425), [426](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L426). 

*/src/vault/VaultController.sol*
*  [5](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L5), [87](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L87), [477](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L477), [604](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L604), [630](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L630). 

*/src/vault/adapter/abstracts/AdapterBase.sol*
*  [6](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L6), [7](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L7), [52](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L52), [262](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L262), [269](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L269), [436](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L436), [465](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L465). 

## 6. Order of Modifiers Not Compliant With Solidity Docs

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#function-declaration) suggests the following modifer order:  Visibility, Mutability, Virtual, Override, Custom modifiers.

*/src/vault/adapter/abstracts/AdapterBase.sol*

* [`_deposit`](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L147) ('internal', 'nonReentrant', 'virtual', 'override'): override (Override) positioned after nonReentrant (Custom Modifiers).

## 7. Contracts Missing @title Tag

21 out of 35 of the contracts in scope are missing a `@title` tag. Given that 14 contracts all have a `@title` tag, consider adding one per the 21 remaining contracts.

[EIP165.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/EIP165.sol), [OnlyStrategy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/OnlyStrategy.sol), [WithRewards.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/WithRewards.sol), [IEIP165.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IEIP165.sol), [IAdminProxy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IAdminProxy.sol), [IWithRewards.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IWithRewards.sol), [ICloneFactory.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ICloneFactory.sol), [IStrategy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IStrategy.sol), [ICloneRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ICloneRegistry.sol), [IPermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IPermissionRegistry.sol), [IVaultRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVaultRegistry.sol), [IYearn.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/IYearn.sol), [IAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IAdapter.sol), [IDeploymentController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IDeploymentController.sol), [ITemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ITemplateRegistry.sol), [IMultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol), [IERC4626.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IERC4626.sol), [IBeefy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/IBeefy.sol), [IMultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol), [IVault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol), and [IVaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVaultController.sol) are missing a `@title` tag.

## 8. Inconsistent Comment Initial Spacing

Almost all comments have an initial space after `//` or `///` while one does not. It is best for code clearity to keep a consistent style.

1. The following contracts only have initial space comments (EX. `// foo`): [EIP165.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/EIP165.sol), [OnlyStrategy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/OnlyStrategy.sol), [AdminProxy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/AdminProxy.sol), [WithRewards.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/WithRewards.sol), [CloneFactory.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneFactory.sol), [PermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol), [CloneRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol), [VaultRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol), [DeploymentController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol), [TemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol), [YearnAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol), [MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol), [BeefyAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol), [MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol), [Vault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol), [VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol), [AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol), [IEIP165.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IEIP165.sol), [IAdminProxy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IAdminProxy.sol), [IWithRewards.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IWithRewards.sol), [ICloneFactory.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ICloneFactory.sol), [IStrategy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IStrategy.sol), [ICloneRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ICloneRegistry.sol), [IPermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IPermissionRegistry.sol), [IVaultRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVaultRegistry.sol), [IYearn.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/IYearn.sol), [IAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IAdapter.sol), [IDeploymentController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IDeploymentController.sol), [ITemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ITemplateRegistry.sol), [IMultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol), [IERC4626.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IERC4626.sol), [IMultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol), [IVault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol), and [IVaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVaultController.sol).
2. No contracts have only no initial space comments (EX. `//foo`).
3. The following contracts have both: [IBeefy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/IBeefy.sol).

## 9. Inconsistent Named Returns

Some functions use named returns and others do not. It is best for code clearity to keep a consistent style.

1. The following contracts only have named returns (EX. `returns(uint256 foo)`): [AdminProxy.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/AdminProxy.sol), [CloneFactory.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneFactory.sol), and [DeploymentController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol).
2. The following contracts only have non-named returns (EX. `returns(uint256)`): [EIP165.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/EIP165.sol), [WithRewards.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/WithRewards.sol), [PermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol), [CloneRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol), [VaultRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol), [TemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol), [YearnAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol), [MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol), and [BeefyAdapter.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol).
3. The following contracts have both: [MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol), [Vault.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol), [VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol), and [AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol).

## 10. Spelling Mistakes

Consider fixing all spelling mistakes.

*/src/vault/AdminProxy.sol*

* The word `Owns` is misspelled as [`ownes`](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/AdminProxy.sol#L11).

## 11. Inconsistent Header Centering

The headers in the codebase are not centered. For example, [this line](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneFactory.sol#L19) is not centered with [this line](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneFactory.sol#L26). Consider centering all headers.

## 12. Low Coverage

The code coverage of the files in scope is not 100%. Consider a test coverage of 100%.

## 13. Inconsistent Pragma Spacing

Some contract in the codebase have a space after the license and before the pragma, but some do not. The following contracts do not have a space, all else do:

* [VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol), [IEIP165.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/IEIP165.sol), [IVaultRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVaultRegistry.sol), [ITemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/ITemplateRegistry.sol), [IMultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/IMultiRewardEscrow.sol), [IERC4626.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IERC4626.sol), [IMultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/IMultiRewardStaking.sol), [IVault.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVault.sol).

## 14. bytes.concat() Can Be Used Over abi.encodePacked()

Consider using `bytes.concat()` instead of `abi.encodePacked()` in contracts with Solidity version >= 0.8.4.

*/src/utils/MultiRewardEscrow.sol*

```solidity
104:	bytes32 id = keccak256(abi.encodePacked(token, account, amount, nonce));
```

*/src/utils/MultiRewardStaking.sol*

```solidity
49:	_name = string(abi.encodePacked("Staked ", IERC20Metadata(address(_stakingToken)).name()));
50:	_symbol = string(abi.encodePacked("pst-", IERC20Metadata(address(_stakingToken)).symbol()));
461:	abi.encodePacked(
```

*/src/vault/Vault.sol*

```solidity
94:	abi.encodePacked("Popcorn", name(), block.timestamp, "Vault")
680:	abi.encodePacked(
```

## 15. Inconsistent Comment Capitalization

There is an inconsistency in the capitalization of comments. For example, [this line](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultRegistry.sol#L27) is not capitilized while [this line](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L66) is. Consider capitilizing all comments.

## 16. Named Returns Not Returning Named Variable

Using named returns may help developers understand what is being returned, but this should be in a `@return` tag. There is no need to confuse developers.

*/src/vault/AdminProxy.sol*

* [`execute`](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L22)

*/src/vault/adapter/abstracts/AdapterBase.sol*

* [`_convertToShares`](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L385)

## 17. Inconsistent Trailing Period

There is a general inconsistency with puncuation in comments, specifically trailing  `.`s. The following items are proof by example and not every case in the codebase:

* [This](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/OnlyStrategy.sol#L6) `@notice` tag does not have a trailing `.` but [this](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L11) does.

* [This](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L34) `@param` tag does not have a trailing `.` but [this](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L35) does. 

* [This](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L262) single line comment does not have a trailing `.` but [this](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L295) does.

## 18. Confusing Word shortening

It seems as though the word `LOGIC` is shortened to [`LOGC`](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L435). Consider spelling the word in full to dilute possible confusion given it is only a one characer difference.

## 19. Inconsistent Headers

Not all headers follow the same general format. Consider sticking to the same format.

**Example [1](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L366):**

```solidity
/*//////////////////////////////////////////////////////////////
                      REWARDS ACCRUAL LOGIC
    //////////////////////////////////////////////////////////////*/
```

**Example [2](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVault.sol#L68):**

```solidity
// MANAGEMENT FUNCTIONS - FEES
```

## 20. Lack of Overflow Error Message

In the [lock](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L88) function of [MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol) [L114](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L114) can overflow (revert) without a proper error message. Consider adding an error message for when a user inputs a large `offset`.

## 21. Unused Cache Variable

The variable `highWaterMark_` in the function [`accruedPerformanceFee`](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L447) appears to be a cache variable that is never used. When returning from the function `highWaterMark` is used instead of `highWaterMark_`. Consider using `highWaterMark_` or removing it.

## 22. Add view Modifier

A view modifier can be added to the following functions to get rid of compiler warnings: 

* [`_calcRewardsEnd`](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L351), [`_encodeAdapterData`](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L242), [`_verifyCreatorOrOwner`](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L667).

## 23. Unused Param

[`_registerVault`](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L390) has a parameter `address vault` that is never used. Consider attending to any security issues that may arrise from the non-use, and possibly removing the parameter.

## 24. Assumptive Language

In [Vault.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol) the phrase `rage quit` is used ([ex](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L616)) which seems to be in jest; however, it is not good to use assumptive language. For individuals that are unaware of the jestful nature (EX. non-native English speakers, etc.) who may also be docile, the assumption that quitting is due to "rage" may be subliminally damaging to ones initial intent with the system.

## 25. Lack of Interface Documentation

It is best practice to document interfaces just like you would core contracts. Most interfaces do not have much internal documentation (comments). For example: [IEIP165.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/IEIP165.sol), [IAdminProxy.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IAdminProxy.sol), [IWithRewards.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IWithRewards.sol), etc. are missing all comments aside from the license identifiers. Consider documenting all interfaces to the level you would core contracts.