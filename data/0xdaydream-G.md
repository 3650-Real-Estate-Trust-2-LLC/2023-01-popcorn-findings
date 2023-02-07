# Gas optimizations
|        |                                           Issue                                           | Instances | Total Gas Saved |
|:------:|:-----------------------------------------------------------------------------------------:|:---------:|:---------------:|
| [G-01] | `++i` costs less gas than `i++`, especially when it’s used in for loops (`--i`/`i--` too) |     1     |        5        |
| [G-02] | Using private rather than public for constants, saves gas                                 |     5     |        -        |
| [G-03] | Functions guaranteed to revert when called by normal users can be marked `payable`        |     13    |       168       |
| [G-04] | Use assembly to check for address(0)                                                      |     4     |        -        |
| [G-05] | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables                    |     8     |       104       |
Total 36 instances with 277 gas saved.
### `++i` costs less gas than `i++`, especially when it’s used in for loops (`--i`/`i--` too)

Saves **5 gas per loop**

_There is 1 instance of this issue:_
```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol
659:    nonces[owner]++,
```

[](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L659)[https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L659](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L659)

### Using private rather than public for constants, saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table.

_There are 5 instances of this issue:_

```solidity
File: src/vault/VaultController.sol
36:    bytes32 public immutable VAULT = "Vault";
37:    bytes32 public immutable ADAPTER = "Adapter";
38:    bytes32 public immutable STRATEGY = "Strategy";
39:    bytes32 public immutable STAKING = "Staking";
```

[](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L36)[https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L36](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L36)

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol
31:    uint256 public constant BPS_DENOMINATOR = 10_000;
```

[](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L31)[https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L31](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L31)

### Functions guaranteed to revert when called by normal users can be marked `payable`

Functions guaranteed to revert when called by normal users can be marked `payable` If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are `CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost

_There are 13 instances of this issue_:

```solidity
File: src/utils/MultiRewardStaking.sol
243:    function addRewardToken(
244:      IERC20 rewardToken,
245:      uint160 rewardsPerSecond,
246:      uint256 amount,
247:      bool useEscrow,
248:      uint192 escrowPercentage,
249:      uint32 escrowDuration,
250:      uint32 offset
251:    ) external onlyOwner {
```

[](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L243)[https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L243](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L243)

```solidity
File: src/vault/AdminProxy.sol
21:    function execute(address target, bytes calldata callData)
20:      external
21:      onlyOwner
```

[](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/AdminProxy.sol#L21)[https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/AdminProxy.sol#L21](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/AdminProxy.sol#L21)

```solidity
File: src/vault/CloneRegistry.sol
41:    function addClone(
42:      bytes32 templateCategory,
43:      bytes32 templateId,
44:      address clone
45:    ) external onlyOwner {
```

[](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneRegistry.sol#L41)[https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneRegistry.sol#L41](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneRegistry.sol#L41)

```solidity
File: src/vault/DeploymentController.sol
99:    function deploy(
100:      bytes32 templateCategory,
101:      bytes32 templateId,
102:      bytes calldata data
103:    ) external onlyOwner returns (address clone) {
```

[](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L99)[https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L99](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L99)

```solidity
File: src/vault/TemplateRegistry.sol
67:    function addTemplate(
68:      bytes32 templateCategory,
69:      bytes32 templateId,
70:      Template memory template
71:    ) external onlyOwner {
```

[](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L67)[https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L67](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L67)

```solidity
File: src/vault/VaultController.sol
579:    function toggleTemplateEndorsements(bytes32[] calldata templateCategories, bytes32[] calldata templateIds)
580:      external
581:      onlyOwner
```

[](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L579)[https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L579](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L579)

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol
55:    function __AdapterBase_init(bytes memory popERC4626InitData)
56:      internal
57:      onlyInitializing
456:    function strategyDeposit(uint256 amount, uint256 shares)
457:      public
458:      onlyStrategy
467:    function strategyWithdraw(uint256 amount, uint256 shares)
468:      public
469:      onlyStrategy
500:    function setHarvestCooldown(uint256 newCooldown) external onlyOwner {
549:    function setPerformanceFee(uint256 newFee) public onlyOwner {
574:    function pause() external onlyOwner {
582:    function unpause() external onlyOwner {
```
[](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L55)[https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L55](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L55)
### Use assembly to check for `address(0)`

_Saves 6 gas per instance_
*There are 4 instances of this issue:*
```solidity
File: src/utils/MultiRewardEscrow.sol
95:    if (token == IERC20(address(0))) revert ZeroAddress();
```

[](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L95)[https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L95](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L95)

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol
83:    if (_strategy != address(0)) _verifyAndSetupStrategy(_requiredSigs);
440:    address(strategy) != address(0) &&
670:    if (recoveredAddress == address(0) || recoveredAddress != owner)
```
[](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L83)[https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L83](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L83)

### `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
*There are 8 instances of this issue:*
```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol
158:    underlyingBalance += _underlyingBalance() - underlyingBalance_;
225:    underlyingBalance -= underlyingBalance_ - _underlyingBalance();
```
[](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L158)[https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L158](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L158)
```solidity
File: src/vault/Vault.sol
228:    shares += feeShares;
357:    amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp
```
[](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L228)[https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L228](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L228)
```solidity
File: src/utils/MultiRewardStaking.sol
408:    rewardInfos[_rewardToken].index += deltaIndex;
431:    accruedRewards[_user][_rewardToken] += supplierDelta;
```
[](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L408)[https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L408](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L408)
```solidity
File: src/utils/MultiRewardEscrow.sol
110:    amount -= fee;
162:    escrows[escrowId].balance -= claimable;
```
[](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L110)[https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L110](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L110)
