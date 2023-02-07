# Summary
## Low Risk
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|L-01       | Missing checks for 0 address|4|
|L-02       | Wrong check for tokenFees|1|
|L-03       | Unspecific compiler version pragma|1|
|L-04       | Return value of call is not checked|1|
|L-05       | Account existence check for low-level calls|1|

## Non critical
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|N-01       | Use indexed fields in an event|18|
|N-02       | Unused return variables|2|
|N-03       | Modifiers should not make state changes|5|

# Details
# Low Risk
## [L-01] Missing checks for 0 address
- [MultiRewardEscrow.sol#L31](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L31)
- [VaultController.sol#L52-L70](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L52-L70)
- [DeploymentController.sol#L35-L44](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L35-L44)
- [VaultController.sol#L53-L70](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L53-L70)

## [L-02] Wrong check for tokenFees
The if statement reverts when the tokenFee is larger than 1e17. However the natspec commment says 1e18 is 100% and I don't see anywhere in the documentation where it says max fee can be 1e17. The if statent should instead check if it's not larger than 1e18.
- [MultiRewardEscrow.sol#L211](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L211)
```solidity
      if (tokenFees[i] >= 1e17) revert DontGetGreedy(tokenFees[i]);
```
## [L-03] Unspecific compiler version pragma
Avoid floating pragmas for non-library contracts.
Contracts should always be deployed with the same compiler version and flags that they have been tested with. Locking the pragma helps ensure that contracts do not accidentally get deployed using a compiler which may have higher risks of undiscovered bugs. 
## [L-04] Return value of call is not checked
The return value of call is not checked in the [execute()](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/AdminProxy.sol#L24) function of `AdminProxy.sol`. This can lead to unexpected failures. 

A recommendation is to check the boolean success and revert the transaction if it's false.
## [L-05] Account existence check for low-level calls
Low-level calls call/delegatecall/staticcall return true even if the account called is non-existent (per EVM design). Account existence must be checked before the calling.
- [AdminProxy.sol#L24](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/AdminProxy.sol#L24)
# Non critical
## [N-01] Use indexed fields in an event
When an event is missing indexed fields it can be hard for off-chain scripts to efficiently filter those events.
- [MultiRewardStaking.sol#L220](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L220)
- [Owned.sol#L38](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/Owned.sol#L38)
- [Owned.sol#L39](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/Owned.sol#L39)
- [OwnedUpgradeable.sol#L40-L41](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/OwnedUpgradeable.sol#L40-L41)
- [CloneRegistry.sol#L33](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneRegistry.sol#L33)
- [PermissionRegistry.sol#L28](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L28)
- [TemplateRegistry.sol#L38-L40](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L38-L40)
- [TemplateRegistry.sol#L88](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L88)
- [Vault.sol#L512-L514](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L512-L514)
- [Vault.sol#L569-L570](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L569-L570)
- [Vault.sol#L621](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L621)
- [VaultController.sol#L741](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L741)
- [VaultController.sol#L781](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L781)
- [VaultController.sol#L824](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L824)
- [VaultController.sol#L853](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L853)
- [VaultRegistry.sol#L36](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L36)
- [AdapterBase.sol#L491](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L491)
- [AdapterBase.sol#L519](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L519)
## [N-02] Unused named return variables
When you have named return variables and you don't use them, then it's better to just specify the type you are gonna return.
- [AdminProxy.sol#L22](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/AdminProxy.sol#L22) success and returndata are never used
- [AdapterBase.sol#L385](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L385) shares is never used
## [N-03] Modifiers should not make state changes
Modifiers should only implement checks and not make state changes. This violates the [checks-effects-interactions-pattern](https://docs.soliditylang.org/en/develop/security-considerations.html#use-the-checks-effects-interactions-pattern).

- The modifier [Vault/takeFees](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L494) calls the `_mint` function which makes state changes. Additonally it also changes the state variables `highWaterMark` and `feesUpdatedAt`.
- The modifier [syncFeeCheckpoint](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L496-L499) changes the state variable `highWaterMark`
- The modifier [accrueRewards](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L371-L384) calls multiple function that changes state variables.
- The modifier [AdapterBase/takeFees](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L559-L567) changes the state variable `highWaterMark` and calls the `_mint` function.