## [G-01] Use assembly to write storage values

Example :
```
    function updateOwner(address newOwner) public {
        owner = newOwner;
    }

    function assemblyUpdateOwner(address newOwner) public {
        assembly {
            sstore(owner.slot, newOwner)
        }
    }
```
*There are 6 instances of this issue*
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L31
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L49
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L50
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L51
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L55
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L56

---
## [G-02] Pack structs

When creating structs, make sure that the variables are listed in ascending order by data type. The compiler will pack the variables that can fit into one 32 byte slot. If the variables are not listed in ascending order, the compiler may not pack the data into one slot, causing additional `sload` and `sstore` instructions when reading/storing the struct into the contract's storage.

*There is 1 instance of this issue*
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/IMultiRewardEscrow.sol#L7

---
## [G-03] Mark storage variables as `immutable` if they never change after contract initialization.

State variables can be declared as constant or immutable. In both cases, the variables cannot be modified after the contract has been constructed. For constant variables, the value has to be fixed at compile-time, while for immutable, it can still be assigned at construction time.

The compiler does not reserve a storage slot for these variables, and every occurrence is inlined by the respective value.

Compared to regular state variables, the gas costs of constant and immutable variables are much lower. For a constant variable, the expression assigned to it is copied to all the places where it is accessed and also re-evaluated each time. This allows for local optimizations. Immutable variables are evaluated once at construction time and their value is copied to all the places in the code where they are accessed. For these values, 32 bytes are reserved, even if they would fit in fewer bytes. Due to this, constant values can sometimes be cheaper than immutable values.


*There is 1 instance of this issue*
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L191

---
## [G-04] Tightly pack storage variables

When defining storage variables, make sure to declare them in ascending order, according to size. When multiple variables are able to fit into one 256 bit slot, this will save storage size and gas during runtime. For example, if you have a bool, uint256 and a bool, instead of defining the variables in the previously mentioned order, defining the two boolean variables first will pack them both into one storage slot since they only take up one byte of storage.

*There are 2 instances of this issue*
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L27
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L26

---
## [G-05] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. Overflow/underflow can be checked in assembly to ensure safety.

Example :
```
    //addition in Solidity
    function addTest(uint256 a, uint256 b) public pure {
        uint256 c = a + b;
    }

    //addition in assembly
    function addAssemblyTest(uint256 a, uint256 b) public pure {
        assembly {
            let c := add(a, b)

            if lt(c, a) {
                mstore(0x00, "overflow")
                revert(0x00, 0x20)
            }
        }
    }
```

*There are 12 instances of this issue*
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L114
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L119
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L181
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L182
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L128
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L198
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L357
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L359
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L393
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L395
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L398
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L424

---
## [G-06] Use assembly to hash instead of Solidity

Example :
```
function solidityHash(uint256 a, uint256 b) public view {
        //unoptimized
        keccak256(abi.encodePacked(a, b));
    }

function assemblyHash(uint256 a, uint256 b) public view {
        //optimized
        assembly {
            mstore(0x00, a)
            mstore(0x20, b)
            let hashedVal := keccak256(0x00, 0x40)
        }
    }
```

*There are 8 instances of this issue*
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L104
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L460
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L464
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L466
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L493
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L495
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L496
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L497

---
## [G-07] Optimal Comparison

When comparing integers, it is cheaper to use strict > & < operators over >= & <= operators, even if you must increment or decrement one of the operands. Note: before using this technique, it's important to consider whether incrementing/decrementing one of the operators could result in an over/underflow. This optimization is applicable when the optimizer is turned off.

*There is 1 instance of this issue*
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L211

---
## [G-08] ++i/i++ should be unchecked{++I}/unchecked{I++} when it is not possible for them to overflow

Solidity version 0.8+ checks for overflow/underflow by default, if overflow/underflow isn’t possible, gas can be saved by using an unchecked block.

*There are 7 instances of this issue*
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L53
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L102
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L155
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L210
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L171
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L373
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L470

---
## [G-09] Mark storage variables as constant if they never change.

State variables can be declared as constant or immutable. In both cases, the variables cannot be modified after the contract has been constructed. For constant variables, the value has to be fixed at compile-time, while for immutable, it can still be assigned at construction time.

The compiler does not reserve a storage slot for these variables, and every occurrence is inlined by the respective value.

Compared to regular state variables, the gas costs of constant and immutable variables are much lower. For a constant variable, the expression assigned to it is copied to all the places where it is accessed and also re-evaluated each time. This allows for local optimizations. Immutable variables are evaluated once at construction time and their value is copied to all the places in the code where they are accessed. For these values, 32 bytes are reserved, even if they would fit in fewer bytes. Due to this, constant values can sometimes be cheaper than immutable values.

*There are 6 instances of this issue*
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L517
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L36
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L37
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L38
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L39
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L40
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L467

---
## [G-10] Setting the constructor to payable 

Making the constructor to payable can eliminate the check for `msg.value`, saving 13 gas on deployment

*There are 9 instances of this issue*
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/AdminProxy.sol#L16
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L20
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L21
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneRegistry.sol#L22
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L24
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneFactory.sol#L23
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/MultiRewardEscrow.sol#L30
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L35
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L53
