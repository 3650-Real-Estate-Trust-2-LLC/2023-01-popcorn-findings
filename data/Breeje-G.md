## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW | 22 |
| [GAS-2](#GAS-2) | X += Y COSTS MORE GAS THAN X = X + Y FOR STATE VARIABLES | 12 |
| [GAS-3](#GAS-3) | NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS | 2 |


### [GAS-1] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop.

*Instances (22):*
```solidity
File: src/vault/VaultController.sol

319:      for (uint8 i = 0; i < len; i++) {

337:      for (uint8 i = 0; i < len; i++) {

357:      for (uint8 i = 0; i < len; i++) {

374:      for (uint8 i = 0; i < len; i++) {

437:      for (uint256 i = 0; i < len; i++) {

494:      for (uint256 i = 0; i < len; i++) {

523:      for (uint256 i = 0; i < len; i++) {

564:      for (uint256 i = 0; i < len; i++) {

587:      for (uint256 i = 0; i < len; i++) {

607:      for (uint256 i = 0; i < len; i++) {

620:      for (uint256 i = 0; i < len; i++) {

633:      for (uint256 i = 0; i < len; i++) {

646:      for (uint256 i = 0; i < len; i++) {

766:      for (uint256 i = 0; i < len; i++) {

806:      for (uint256 i = 0; i < len; i++) {

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol)

```solidity
File: src/vault/PermissionRegistry.sol

42:      for (uint256 i = 0; i < len; i++) {

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L42)

```solidity
File: src/utils/MultiRewardEscrow.sol

53:      for (uint256 i = 0; i < escrowIds.length; i++) {

102:     nonce++;

155:     for (uint256 i = 0; i < escrowIds.length; i++) {

210:     for (uint256 i = 0; i < tokens.length; i++) {

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol)

```solidity
File: src/utils/MultiRewardStaking.sol

171:      for (uint8 i; i < _rewardTokens.length; i++) {

373:      for (uint8 i; i < _rewardTokens.length; i++) {

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol)

### [GAS-2] X += Y COSTS MORE GAS THAN X = X + Y FOR STATE VARIABLES

Same goes for subtraction as well. X -= Y Costs more gas than X = X - Y.

*Instances (12):*
```solidity
File: src/utils/MultiRewardStaking.sol

110:      amount -= fee;

162:      escrows[escrowId].balance -= claimable;

357:      amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);

408:      rewardInfos[_rewardToken].index += deltaIndex;

431:      accruedRewards[_user][_rewardToken] += supplierDelta;

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol)

```solidity
File: src/vault/Vault.sol

228:      shares += feeShares;

343:      shares += shares.mulDiv(

365:      assets += assets.mulDiv(

387:      assets -= assets.mulDiv(

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol)

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

158:      underlyingBalance += _underlyingBalance() - underlyingBalance_;

225:      underlyingBalance -= underlyingBalance_ - _underlyingBalance();

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol)

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

178:      assets -= assets.mulDiv(

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L178)

### [GAS-3] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

*Instances (2):*
```solidity
File: src/vault/AdminProxy.sol

22:     returns (bool success, bytes memory returndata)

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/AdminProxy.sol#L22)

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

385:     returns (uint256 shares)

```
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L385)