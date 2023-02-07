## GAS-1: ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow

### Description

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

### Affected file

* MultiRewardEscrow.sol (Line: 53, 155, 210)
* MultiRewardStaking.sol (Line: 171, 373)
* PermissionRegistry.sol (Line: 42)
* VaultController.sol (Line: 319, 337, 357, 374, 437, 494, 523, 564, 587, 607, 620, 633, 646, 766, 806)

## GAS-2: <X> += <Y> costs more gas than <X> = <X> + <Y> for state variables

### Description

Using the addition operator instead of plus-equals saves gas.

### Affected file

* MultiRewardEscrow.sol (Line: 162)
* MultiRewardStaking.sol (Line: 408, 431)

## GAS-3: <X> <= <Y> costs more gas than <X> < <Y> + 1

### Description

In Solididy, the opcode 'less or equal' doesn't exist. So the EVM will translate this by two distinct operation, first the inferior, and then the equal which cost more gas then a strict less.

### Affected file

* MultiRewardEscrow.sol (Line: 211)
* Vault.sol (Line: 527, 528, 529, 530)

## GAS-4: Empty blocks should be removed or emit something

### Description

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

### Affected file

* AdminProxy.sol (Line: 16)
* CloneFactory.sol (Line: 23)
* CloneRegistry.sol (Line: 22)
* EIP165.sol (Line: 7)
* PermissionRegistry.sol (Line: 20)
* TemplateRegistry.sol (Line: 24)
* Vault.sol (Line: 473)
* VaultRegistry.sol (Line: 21)
* WithRewards.sol (Line: 13, 15)

## GAS-5: Internal functions not called by the contract should be removed to save deployment gas

### Description

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword.

### Affected file

* BeefyAdapter.sol (Line: 108, 117, 192, 203)
* MultiRewardStaking.sol (Line: 98, 102, 107, 121, 137)
* YearnAdapter.sol (Line: 80, 84, 158, 166)

## GAS-6: Internal functions only called once can be inlined to save gas

### Description

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

### Affected file

* MultiRewardStaking.sol (Line: 191)
* VaultController.sol (Line: 120, 141, 154, 225, 242, 256, 390, 692)
* YearnAdapter.sol (Line: 89, 101, 109, 114)

## GAS-7: Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

### Description

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

### Affected file

* MultiRewardEscrow.sol (Line: 67, 69)
* MultiRewardStaking.sol (Line: 216, 218, 440)
* VaultRegistry.sol (Line: 28, 31)

## GAS-8: Not using the named return variables when a function returns, wastes deployment gas

### Description

It is not necessary to have both a named return and a return statement.

### Affected file

* AdminProxy.sol (Line: 23)

## GAS-9: Public functions not called by the contract should be declared external instead

### Description

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

### Affected file

* BeefyAdapter.sol (Line: 148, 167, 221, 230)
* EIP165.sol (Line: 7)
* MultiRewardStaking.sol (Line: 445)
* Vault.sol (Line: 664)
* WithRewards.sol (Line: 15, 21)
* YearnAdapter.sol (Line: 144)

## GAS-10: Replace modifier with function

### Description

Modifiers make code more elegant, but cost more than normal functions.

### Affected file

* MultiRewardStaking.sol (Line: 371)
* OnlyStrategy.sol (Line: 10)
* Vault.sol (Line: 480, 496)
* VaultController.sol (Line: 704)

## GAS-11: Should use arguments instead of state variable

### Description

Using function's parameter cost less gas then a state variable.

### Affected file

* Vault.sol (Line: 635)

## GAS-12: Storage pointer to a structure is cheaper than copying each value of the structure into memory, same for array and mapping

### Description

It may not be obvious, but every time you copy a storage struct/array/mapping to a memory variable, you are literally copying each member by reading it from storage, which is expensive. And when you use the storage keyword, you are just storing a pointer to the storage, which is much cheaper.

### Affected file

* DeploymentController.sol (Line: 104)

## GAS-13: Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

### Description

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

### Affected file

* MultiRewardStaking.sol (Line: 171, 373, 404)
* VaultController.sol (Line: 314, 319, 336, 337, 353, 357, 373, 374, 436, 488, 517, 563, 583, 606, 619, 632, 645, 765, 805)

## GAS-14: abi.encode() is less efficient than abi.encodePacked()

### Description

Use abi.encodePacked() where possible to save gas.

### Affected file

* MultiRewardStaking.sol (Line: 465, 494)
* Vault.sol (Line: 684, 719)
* VaultController.sol (Line: 219)