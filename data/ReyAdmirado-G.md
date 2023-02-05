| | issue |
| ----------- | ----------- |
| 1 | [State variables only set in the constructor should be declared immutable.](#1-state-variables-only-set-in-the-constructor-should-be-declared-immutable) |
| 2 | [Structs can be packed into fewer storage slots](#2-structs-can-be-packed-into-fewer-storage-slots) |
| 3 | [state variables can be packed into fewer storage slots](#3-state-variables-can-be-packed-into-fewer-storage-slots) |
| 4 | [Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate](#4-multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct-where-appropriate) |
| 5 | [state variables should be cached in stack variables rather than re-reading them from storage](#5-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage-ones-not-caught-by-c4udit) |
| 6 | [Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if` statement](#6-add-unchecked--for-subtractions-where-the-operands-cannot-underflow-because-of-a-previous-require-or-if-statement) |
| 7 | [`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables](#7-x--y-costs-more-gas-than-x--x--y-for-state-variables) |
| 8 | [not using the named return variables when a function returns, wastes deployment gas](#8-not-using-the-named-return-variables-when-a-function-returns-wastes-deployment-gas) |
| 9 | [can make the variable outside the loop to save gas](#9-can-make-the-variable-outside-the-loop-to-save-gas) |
| 10 | [`++i/i++` should be `unchecked{++i}/unchecked{i++}` when it is not possible for them to overflow, as is the case when used in for-loop and while-loops](#10-ii-should-be-uncheckediuncheckedi-when-it-is-not-possible-for-them-to-overflow-as-is-the-case-when-used-in-for-loop-and-while-loops) |
| 11 | [use a more recent version of solidity](#11-use-a-more-recent-version-of-solidity) |
| 12 | [internal functions only called once can be inlined to save gas](#12-internal-functions-only-called-once-can-be-inlined-to-save-gas) |
| 13 | [abi.encode() is less efficient than abi.encodepacked()](#13-abiencode-is-less-efficient-than-abiencodepacked) |
| 14 | [usage of uint/int smaller than 32 bytes (256 bits) incurs overhead](#14-usage-of-uintint-smaller-than-32-bytes-256-bits-incurs-overhead) |
| 15 | [ Ternary over if ... else](#15-ternary-over-if--else) |
| 16 | [public functions not called by the contract should be declared external instead](#16-public-functions-not-called-by-the-contract-should-be-declared-external-instead) |
| 17 | [should use arguments instead of state variable](#17-should-use-arguments-instead-of-state-variable) |
| 18 | [instead of calculating a statevar with keccak256() every time the contract is made pre calculate them before and only give the result to a constant](#18-instead-of-calculating-a-statevar-with-keccak256-every-time-the-contract-is-made-pre-calculate-them-before-and-only-give-the-result-to-a-constant-or-immutable) |
| 19 | [use existing cached version of state var](#19-use-existing-cached-version-of-state-var) |
| 20 | [Optimize names to save gas](#20-optimize-names-to-save-gas) |
| 21 | [Non-strict inequalities are cheaper than strict ones](#21-non-strict-inequalities-are-cheaper-than-strict-ones) |
| 22 | [Setting the constructor to payable](#22-setting-the-constructor-to-payable) |
| 23 | [instead of assigning to zero use `delete`](#23-instead-of-assigning-to-zero-use-delete) |
| 24 | [part of the code can be pre calculated](#24-part-of-the-code-can-be-pre-calculated) |

## 1. State variables only set in the constructor should be declared immutable.

Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmaccess (100 gas) with a PUSH32 (3 gas).

`feeRecipient` can be declared immutable
- [MultiRewardEscrow.sol#L191](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L191)


## 2. Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings

put `account` near the `start` so the 3 uint32 variables(12 byte) and the address variable(20 bytes) be in the same slot(32byte). this will save one slot usage
- [IMultiRewardEscrow.sol#L7-L22](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardEscrow.sol#L7-L22)


## 3. state variables can be packed into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables are also cheaper.

`_decimals`(#L38) and `feeRecipient`(#L510) can be put together in the same slot if they be declared near each other. there is no reads that uses both variable so mainly saves Gsset of 20000 for a slot. bring the `feeRecipient` near `_decimals`
- [Vault.sol#L38](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L38)


## 4. Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. 

`templateExists` and `templateCategoryExists` can be simplified to use less slots. used together once so it will save a bit gas there too.
- [TemplateRegistry.sol#L33-L35](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L33-L35)


## 5. state variables should be cached in stack variables rather than re-reading them from storage (ones not caught by c4udit)

Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

possible 97 gas save with risking 3 gas. if the check fails we save 97 for the second `quitPeriod` read and if it passes we just lose 3 gas. cache it before this line
- [Vault.sol#L541](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L541)

possible 97 gas save with risking 3 gas. if the check fails we save 97 for the second `quitPeriod` read and if it passes we just lose 3 gas. cache it before this line
- [Vault.sol#L595](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L595)

caching `deploymentController` will save 97 gas. its being used twice in #L837 and #L840 and can be cached before it.
- [VaultController.sol#L837-L840](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L837-L840)

cache `yVault.totalSupply()` before the if statement because it will usually be false and we will save big gas for a risk of losing 3 gas if the check be true. used twice in #L90 and #L95. should be cached before this line
- [YearnAdapter.sol#L90](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L90)

cache `VaultAPI(IYearnRegistry(externalRegistry).latestVault(_asset))` and assign the cached version to `yVault` in #L45 so we can use the cached version in #L54 to save gas.
- [YearnAdapter.sol#L45-L54](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L45-L54)


### (the next two will be the complete version of the c4audit finding because its not completely marked there)
caching `adapter` will save 97 gas. `adapter` is being used twice in #L604 and #L606, we can reduce one SLOAD with caching it before #L604.(it changes after #L608 so the #L610 `adapter` should be cached version of `proposedAdapter` that I explain after this one)
- [Vault.sol#L604](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L604)

the `proposedAdapter` is being used in both #L606 and #L608 and can be cached before to save gas as c4udit found, but `adapter` in line 610 can be the cached version of `proposedAdapter` instead. this will save 100 more gas. cache before #L606
- [Vault.sol#L606](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L606)


## 6. Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if` statement

require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
this will stop the check for overflow and underflow so it will save gas

- [YearnAdapter.sol#L151](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L151)
- [MultiRewardStaking.sol#L357](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L357)
- [MultiRewardStaking.sol#L392](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L392)
- [MultiRewardStaking.sol#L394](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L394)


## 7. `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using the addition operator instead of plus-equals saves gas

- [MultiRewardStaking.sol#L408](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L408)
- [MultiRewardStaking.sol#L431](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L431)

- [MultiRewardEscrow.sol#L162](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L162)


## 8. not using the named return variables when a function returns, wastes deployment gas

- [AdminProxy.sol#L22](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L22)


## 9. can make the variable outside the loop to save gas

make the variable outside and only give the value to variable inside

`success` and `returnData`
- [VaultController.sol#L323](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L323)
- [VaultController.sol#L338](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L338)
- [VaultController.sol#L360](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L360)
- [VaultController.sol#L375](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L375)
- [VaultController.sol#L450](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L450)
- [VaultController.sol#L497](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L497)
- [VaultController.sol#L565](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L565)
- [VaultController.sol#L588](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L588)
- [VaultController.sol#L609](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L609)
- [VaultController.sol#L622](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L622)
- [VaultController.sol#L635](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L635)
- [VaultController.sol#L648](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L648)
- [VaultController.sol#L767](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L767)
- [VaultController.sol#L807](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L807)

7 varibles here
- [VaultController.sol#L439-L445](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L439-L445)

`escrowId` and `escrow` and `claimable`
- [MultiRewardEscrow.sol#L156-L159](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L156-L159)

`rewardAmount` and `escrowInfo`
- [MultiRewardStaking.sol#L172-L176](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L172-L176)

`rewardToken` and `rewards`
- [MultiRewardStaking.sol#L374-L375](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L374-L375)


## 10. `++i/i++` should be `unchecked{++i}/unchecked{i++}` when it is not possible for them to overflow, as is the case when used in for-loop and while-loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

- [PermissionRegistry.sol#L42](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L42)

- [VaultController.sol#L319](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L319)
- [VaultController.sol#L337](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L337)
- [VaultController.sol#L357](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L357)
- [VaultController.sol#L374](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L374)
- [VaultController.sol#L437](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L437)
- [VaultController.sol#L494](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L494)
- [VaultController.sol#L523](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L523)
- [VaultController.sol#L564](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L564)
- [VaultController.sol#L587](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L587)
- [VaultController.sol#L607](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L607)
- [VaultController.sol#L620](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L620)
- [VaultController.sol#L633](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L633)
- [VaultController.sol#L646](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L646)
- [VaultController.sol#L766](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L766)
- [VaultController.sol#L806](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L806)

- [MultiRewardEscrow.sol#L53](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L53)
- [MultiRewardEscrow.sol#L155](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L155)
- [MultiRewardEscrow.sol#L210](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L210)

- [MultiRewardStaking.sol#L171](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L171)
- [MultiRewardStaking.sol#L373](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L373)


## 11. use a more recent version of solidity

use a strict version of solidity

Use a solidity version of at least 0.8.17 to  get prevention of the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.


## 12. internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

`_registerCreatedVault`
- [VaultController.sol#L141](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L141)

`_handleVaultStakingRewards`
- [VaultController.sol#L154](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L154)

`__deployAdapter`
- [VaultController.sol#L225](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L225)

`_encodeAdapterData`
- [VaultController.sol#L242](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L242)

`_deployStrategy`
- [VaultController.sol#L256](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L256)

`_registerVault`
- [VaultController.sol#L390](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L390)

`_verifyAdapterConfiguration`
- [VaultController.sol#L692](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L692)

`_shareValue`
- [YearnAdapter.sol#L89](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L89)

`_freeFunds`
- [YearnAdapter.sol#L101](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L101)

`_yTotalAssets`
- [YearnAdapter.sol#L109](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L109)

`_calculateLockedProfit`
- [YearnAdapter.sol#L114](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L114)

`_lockToken`
- [MultiRewardStaking.sol#L191](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L191)


## 13. abi.encode() is less efficient than abi.encodepacked()


- [VaultController.sol#L219](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L219)


## 14. usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Use a larger size then downcast where needed

when using smaller versions of types like uint or bytes wouldnt help us to send stuff in one slot(like for  a struct) its better to just use the 256 bits version of types to not waste gas.

- [WithRewards.sol#L21](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/WithRewards.sol#L21)

- [Vault.sol#L38](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L38)

many places for `len` and `i` in
- [VaultController.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol)

- [MultiRewardStaking.sol#L33](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L33)


## 15. Ternary over if ... else

Using ternary operator instead of the if else statement saves gas.

- [YearnAdapter.sol#L118-L125](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L118-L125)

- [MultiRewardStaking.sol#L178-L184](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L178-L184)


## 16. public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

- [WithRewards.sol#L15](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/WithRewards.sol#L15)
- [WithRewards.sol#L21](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/WithRewards.sol#L21)

- [BeefyAdapter.sol#L230](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L230)


## 17. should use arguments instead of state variable

This will save near 97 gas

instead of `asset` `asset_` can be used 
- [Vault.sol#L97](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L97)


## 18. instead of calculating a statevar with keccak256() every time the contract is made pre calculate them before and only give the result to a constant or immutable

`bytes4(keccak256("deploy(bytes32,bytes32,bytes)"))` always has the same answer so it can be pre calculated and only the answer should be given to the immutable or constant so it doesnt get calculated every time the contract is made.
- [VaultController.sol#L40](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L40)


## 19. use existing cached version of state var

saves 97 gas each time

instead of `highWaterMark` use the cached version `highWaterMark_` in these lines. 2 instances
- [Vault.sol#L453-455](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L453-455)


## 20. Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).
you can use this [tool](https://emn178.github.io/solidity-optimize-name/) to get the optimized version for function and properties signitures 

vault.sol has lots of external and public function and lots of properties and it would be much more efficient if the functions and properties been sorted by the most used functions and properties. 
- [Vault.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)


## 21. Non-strict inequalities are cheaper than strict ones

In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).
consider replacing >= with the strict counterpart > and add `- 1` to the second side

4 instances here
- [Vault.sol#L527-L530](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L527-L530)

- [YearnAdapter.sol#L150](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L150)

- [MultiRewardEscrow.sol#L211](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L211)


## 22. Setting the constructor to payable

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

- [VaultController.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol)
- [TemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol)
- [MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol)


## 23. instead of assigning to zero use `delete` 

assigning to zero uses more gas than using `delete` , and they both assign variable to default value. so it is encouraged if the data is no longer needed use delete instead.

- [MultiRewardStaking.sol#L186](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L186)


## 24. part of the code can be pre calculated

the keccak256() used here will always be the same so there is no need to calculate it inside the function. just pre calculate the answer to it and use the answer inside the code instead. use a comment to show what it was to the readers

`keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),`
- [Vault.sol#L685-L687](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L685-L687)

`keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),`
- [Vault.sol#L720-L722](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L720-L722)

`keccak256("1"),`
- [Vault.sol#L724](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L724)

`keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)")`
- [MultiRewardStaking.sol#L466](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L466)

`keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),`
- [MultiRewardStaking.sol#L495](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L495)

`keccak256("1"),`
- [MultiRewardStaking.sol#L497](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L497)
