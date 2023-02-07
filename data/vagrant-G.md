|   | Issue                                                                 | Instances          | Total Gas Saved |
| - | --------------------------------------------------------- | ------------------ | ------------------ |
| [G-01] | Internal functions only called once can be inlined to save gas | 7 | - |
| [G-02] | ++i/i++ Should Be unchecked{++i}/unchecked{i++} When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops | 13 | 455 |
| [G-03] | <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables | 2 | - |
| [G-04] | public functions not called internally can be marked external to save gas | 6 | - |
| [G-05] | Functions guaranteed to revert when called by normal users can be marked payable | 24 | 504 |
| [G-06] | constructor can be marked payable | 8 | 104 |

#### Gas Optimizations 4

### [G-01] Internal functions only called once can be inlined to save gas

PoC:

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L120
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L390
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L141
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L154
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L225
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L242
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L256


###  [G-02] ++i/i++ Should Be unchecked{++i}/unchecked{i++} When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops

PoC:

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L319
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L337
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L357
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L374
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L437
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L494
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L523
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L564
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L587
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L607
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L620
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L633
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L646

### [G-03] <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables

PoC

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L158
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L225

### [G-04] public functions not called internally can be marked external to save gas

The functions marked in the NC-Section of the C4udit output are excluded here, but they can be optimized in the same way. 

PoC:

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L456
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L467
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L549
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L632

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/WithRewards.sol#L15
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/WithRewards.sol#L21

###  [G-05] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

PoC

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L99
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L121

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L525
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L553
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L578
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L629
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L643
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L648

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/AdminProxy.sol#L19

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneFactory.sol#L39

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneRegistry.sol#L41

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L52
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L67
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L102

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L44

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L723
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L751
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L764
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L791
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L804
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L832
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L864

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L500
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L549


### [G-06] Constructor can be marked payable

Saves ~13 gas per instance

PoC

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/AdminProxy.sol
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneFactory.sol
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneRegistry.sol
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol
