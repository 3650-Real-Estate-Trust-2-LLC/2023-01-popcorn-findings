# Summary

## Gas Optimizations

|       | Issue                                                                                                 | Instances |
| ----- | :---------------------------------------------------------------------------------------------------- | :-------: |
| GAS-1 | `<x> += <y>`/`<x> -= <y>` costs more gas than `<x> = <x> + <y>`/`<x> = <x> - <y>` for state variables |    12     |
| GAS-2 | Setting the constructor to payable                                                                    |     9     |
| GAS-3 | DIV BY 0                                                                                              |     2     |
| GAS-4 | DIV BY 0                                                                                              |     6     |

### [GAS-1] `<x> += <y>`/`<x> -= <y>` costs more gas than `<x> = <x> + <y>`/`<x> = <x> - <y>` for state variables

Using compound assignment operators for state variables (like `State += X` or S`tate -= X` …) it’s more expensive than using operator assignment (like `State = State + X` or `State = State - X` …).

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardEscrow.sol

110:       amount -= fee;

162:       escrows[escrowId].balance -= claimable;

```

```solidity
File: src/utils/MultiRewardStaking.sol

357:       amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);

408:     rewardInfos[_rewardToken].index += deltaIndex;

431:     accruedRewards[_user][_rewardToken] += supplierDelta;

```

```solidity
File: src/vault/Vault.sol

228:         shares += feeShares;

343:         shares += shares.mulDiv(

365:         assets += assets.mulDiv(

387:         assets -= assets.mulDiv(

```

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

158:         underlyingBalance += _underlyingBalance() - underlyingBalance_;

225:             underlyingBalance -= underlyingBalance_ - _underlyingBalance();

```

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

178:             assets -= assets.mulDiv(

```

### [GAS-2] Setting the constructor to payable

Saves ~13 gas per instance

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardEscrow.sol

30:   constructor(address _owner, address _feeRecipient) Owned(_owner) {

```

```solidity
File: src/vault/AdminProxy.sol

16:   constructor(address _owner) Owned(_owner) {}

```

```solidity
File: src/vault/CloneFactory.sol

23:   constructor(address _owner) Owned(_owner) {}

```

```solidity
File: src/vault/CloneRegistry.sol

22:   constructor(address _owner) Owned(_owner) {}

```

```solidity
File: src/vault/DeploymentController.sol

35:   constructor(

```

```solidity
File: src/vault/PermissionRegistry.sol

20:   constructor(address _owner) Owned(_owner) {}

```

```solidity
File: src/vault/TemplateRegistry.sol

24:   constructor(address _owner) Owned(_owner) {}

```

```solidity
File: src/vault/VaultController.sol

53:   constructor(

```

```solidity
File: src/vault/VaultRegistry.sol

21:   constructor(address _owner) Owned(_owner) {}

```

### [GAS-3] DIV BY 0

#### Description:

Division by 0 can lead to accidentally revert

##### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardEscrow.sol

181:         (escrow.balance * (block.timestamp - uint256(escrow.lastUpdateTime))) /

```

```solidity
File: src/utils/MultiRewardStaking.sol

359:     return (block.timestamp + (amount / uint256(rewardsPerSecond))).safeCastTo32();

```

### [GAS-4] USE FUNCTION INSTEAD OF MODIFIERS (4 INSTANCES)

#### **Proof Of Concept**

```solidity
File: src/utils/MultiRewardStaking.sol

371:   modifier accrueRewards(address _caller, address _receiver) {

```

```solidity
File: src/vault/Vault.sol

480:     modifier takeFees() {

496:     modifier syncFeeCheckpoint() {

```

```solidity
File: src/vault/VaultController.sol

704:   modifier canCreate() {

```

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

559:     modifier takeFees() {

```

```solidity
File: src/vault/adapter/abstracts/OnlyStrategy.sol

10:   modifier onlyStrategy() {

```