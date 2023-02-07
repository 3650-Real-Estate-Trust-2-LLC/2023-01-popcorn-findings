# SMART CONTRACT AUDIT REPORT


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Missing address zero check when nominating new owner | 2 |
| [GAS-2](#GAS-2) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops, like wise `--i` costs less gas than `i--` | 24 |
| [GAS-3](#GAS-3) | No check for updating state variable to current value | 2 |

### <a name="GAS-1"></a>[GAS-1] Missing address zero check when nominating new owner
when nominating a new owner there is no check for address zero to save the gas that will be used to update the nominateOwner value and emit the OwnerNominated

*Instances (2)*:
```solidity
File: src/vault/Owned.sol

17:   function nominateNewOwner(address _owner) external virtual onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Owned.sol)

```solidity
File: src/vault/DeploymentController.sol

121:   nominateNewDependencyOwner(address _owner) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol)


### <a name="GAS-2"></a>[GAS-2] `++i` costs less gas than `i++`, especially when it's used in `for`-loops, like wise `--i` costs less gas than `i--`
*Saves 5 gas per loop*

*Instances (24)*:
```solidity
File: src/utils/MultiRewardEscrow.sol

53:     for (uint256 i = 0; i < escrowIds.length; i++) {

102:     nonce++;

155:     for (uint256 i = 0; i < escrowIds.length; i++) {

210:     for (uint256 i = 0; i < tokens.length; i++) {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol)

```solidity
File: src/utils/MultiRewardStaking.sol

171:     for (uint8 i; i < _rewardTokens.length; i++) {

373:     for (uint8 i; i < _rewardTokens.length; i++) {

470:                 nonces[owner]++,

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol)

```solidity
File: src/vault/PermissionRegistry.sol

42:     for (uint256 i = 0; i < len; i++) {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol)

```solidity
File: src/vault/Vault.sol

691:                                 nonces[owner]++,

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol)

```solidity
File: src/vault/VaultController.sol

319:     for (uint8 i = 0; i < len; i++) {

337:     for (uint8 i = 0; i < len; i++) {

357:     for (uint8 i = 0; i < len; i++) {

374:     for (uint8 i = 0; i < len; i++) {

437:     for (uint256 i = 0; i < len; i++) {

494:     for (uint256 i = 0; i < len; i++) {

523:     for (uint256 i = 0; i < len; i++) {

564:     for (uint256 i = 0; i < len; i++) {

587:     for (uint256 i = 0; i < len; i++) {

607:     for (uint256 i = 0; i < len; i++) {

620:     for (uint256 i = 0; i < len; i++) {

633:     for (uint256 i = 0; i < len; i++) {

646:     for (uint256 i = 0; i < len; i++) {

766:     for (uint256 i = 0; i < len; i++) {

806:     for (uint256 i = 0; i < len; i++) {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol)

### <a name="GAS-3"></a>[GAS-3] No check for updating state variable to current value
when nominating a new owner there is no Check that the address being passed to the nominateNewOwner is not the the same to the current value of the owner to save the gas that will be used to update the nominateOwner value and emit the OwnerNominated

*Instances (2)*:
```solidity
File: src/vault/Owned.sol

17:   function nominateNewOwner(address _owner) external virtual onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Owned.sol)

```solidity
File: src/vault/DeploymentController.sol

121:   nominateNewDependencyOwner(address _owner) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol)