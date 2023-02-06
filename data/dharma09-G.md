## Summary

**Gas Optimizations**

|  | Issue | Instances |
| --- | --- | --- |
| [G‑01] | Use scientific notation (e.g 1e18) rather than exponentiation (e.g 10**18) | 1 |
| [G‑02] | Using storage to declare Struct variable inside function | 8 |
| [G-03] | Using delete statement can save gas | 2 |
| [G-04] | Use assembly to write address storage values | 2 |
| [G-05] | Using unchecked blocks to save gas | 1 |
| [G-06] | Make Variable constant/immutable   | 9 |
| [G-07] | Vault.accruedPerformanceFee() Use cached value  | 1 |
| [G-08] | Use immutable on variables that are set and never changed after(2.1 K per variable per usage) | 17 |
| [G-9] | ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow,as is the case when used in for-and while-loops | 20 |
| [G-10] | Setting the constructor to payable  | 5 |
| [G-11] | usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead | 3 |

Total: 69 instances over 11 issues

### [G-01] **Use scientific notation `(e.g 1e18)` rather than exponentiation `(e.g 10**18)`**

*There are 1 instances of this issue:*

**Affected source code:**

```solidity
File: /src/vault/adapter/yearn/YearnAdapter.sol
25: uint256 constant DEGRADATION_COEFFICIENT = 10**18;
```

- [YearnAdapter.sol:25](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L25)

### **[G-02] Using `storage` to declare Struct variable inside function**

instead of caching variable to memory. read it directly from storage.

*There are 8 instances of this issue:*

**Affected source code:**

```solidity
File: /src/utils/MultiRewardStaking.sol
176: EscrowInfo memory escrowInfo = escrowInfos[_rewardTokens[i]];
254: RewardInfo memory rewards = rewardInfos[rewardToken];
297: RewardInfo memory rewards = rewardInfos[rewardToken];
328: RewardInfo memory rewards = rewardInfos[rewardToken];
375: RewardInfo memory rewards = rewardInfos[rewardToken];
414: RewardInfo memory rewards = rewardInfos[_rewardToken];

/src/utils/MultiRewardEscrow.sol
157: Escrow memory escrow = escrows[escrowId];

/src/vault/DeploymentController.sol
104: Template memory template = templateRegistry.getTemplate(templateCategory, templateId);
```

- [MultiRewardStaking.sol:176](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L176)
- [DeploymentController.sol:104](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L104)
- [MultiRewardEscrow.sol:157](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L157)

### **[G-03] Using `delete` statement can save gas**

*There are 2 instances of this issue:*

**Affected source code:**

```solidity

File: /src/vault/adapter/abstracts/AdapterBase.sol
577: underlyingBalance = 0;

File: File: src/utils/MultiRewardStaking.sol
186: accruedRewards[user][_rewardTokens[i]] = 0;
```

Mitigation:

```diff
File: /src/Splits.sol
-577: underlyingBalance = 0;
+577: delete underlyingBalance
```

- [AdapterBase.sol:577](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L577)
- [MultiRewardStaking.sol:186](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L186)

### **[G-04] Use `assembly` to write *address storage values***

*There are 2 instances of this issue:*

**Affected source code:**

```solidity
File: /src/utils/MultiRewardEscrow.sol
31: feeRecipient = _feeRecipient;

File: /src/vault/Vault.sol
558: feeRecipient = _feeRecipient;
```

### [G-05] Using unchecked blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block [see resource](https://github.com/ethereum/solidity/issues/10695)

*There are 1 instances of this issue:*

**Affected source code:**

```solidity
File: /src/utils/MultiRewardEscrow.sol
102: nonce++;
```

- [MultiRewardEscrow.sol:102](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L102)

### **[G-06] Make Variable `constant`/`immutable`**

Making variables constant/immutable, if possible, saves gas as all variables get replaced by the values assigned to them.

*There are 9 instances of this issue:*

**Affected source code:**

```solidity
File: /src/vault/Vault.sol
466: uint256 public highWaterMark = 1e18;
619: uint256 public quitPeriod = 3 days;
657: uint256 internal INITIAL_CHAIN_ID;
658: bytes32 internal INITIAL_DOMAIN_SEPARATOR;

File: /src/vault/adapter/abstracts/AdapterBase.sol
557: address FEE_RECIPIENT = address(0x4444);
625: uint256 internal INITIAL_CHAIN_ID;
626: bytes32 internal INITIAL_DOMAIN_SEPARATOR;

File: /src/utils/MultiRewardStaking.sol
438: uint256 internal INITIAL_CHAIN_ID;
439: bytes32 internal INITIAL_DOMAIN_SEPARATOR;
```

### **[G-07] `Vault.accruedPerformanceFee()` : Use cached value**

The code can be optimized by minimizing the number of SLOADs.

*There are 1 instances of this issue:*

**Affected source code:**

```solidity
File: /src/vault/Vault.sol
452:  return
453:            performanceFee > 0 && shareValue > highWaterMark  // see L:448 highWaterMark_
454:                ? performanceFee.mulDiv(
455:                    (shareValue - highWaterMark) * totalSupply(), // see L:448 highWaterMark_
456:                    1e36,
457:                    Math.Rounding.Down
458:                )
459:                : 0;
460:    }
```

- [Vault.sol:453](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L453)

### **[G-08] Use immutable on variables that are set and never changed after(2.1 K per variable per usage)**

*There are 17 instances of this issue:*

**Affected source code:**

```solidity
File: /src/vault/DeploymentController.sol
23: ICloneFactory public cloneFactory;
24: ICloneRegistry public cloneRegistry;
26: ITemplateRegistry public templateRegistry;

File: /src/vault/VaultRegistry.sol
34: address[] public allVaults;
 
File: /src/utils/MultiRewardEscrow.sol
191: address public feeRecipient;

File: /src/vault/VaultController.sol
717: IAdminProxy public adminProxy;
387: IVaultRegistry public vaultRegistry;
822: IPermissionRegistry public permissionRegistry;
535: IMultiRewardEscrow public escrow; 

/src/vault/CloneRegistry.sol
31: address[] public allClones;	

File: /src/utils/MultiRewardStaking.sol
157: IMultiRewardEscrow public escrow;  
208: IERC20[] public rewardTokens;

/src/vault/VaultController.sol
535: IMultiRewardEscrow public escrow
739: uint256 public performanceFee;

File: /src/vault/Vault.sol
467: uint256 public assetsCheckpoint;
468: uint256 public feesUpdatedAt;
565: IERC4626 public adapter;
```

### **[G-09] `++i/i++` should be `unchecked{++i}/unchecked{i++}` when it is not possible for them to overflow,as is the case when used in for-and while-loops**

*Instances (20)*:

```solidity
File: src/utils/MultiRewardEscrow.sol

53:     for (uint256 i = 0; i < escrowIds.*length*; i++) {
155:    for (uint256 i = 0; i < escrowIds.*length*; i++) {
210:    for (uint256 i = 0; i < tokens.*length*; i++) {
```

[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol)

```solidity
File: src/vault/PermissionRegistry.sol
42:     for (uint256 i = 0; i < len; i++) {
```

[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol)

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

```solidity
File: /src/utils/MultiRewardStaking.sol
171: for (uint8 i; i < _rewardTokens.length; i++) {
373: for (uint8 i; i < _rewardTokens.length; i++)
```

[Link to code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L171)

### **[G-10] Setting the `constructor` to `payable`**

Saves ~13 gas per instance

*There are 5 instances of this issue:*

**Affected source code:**

```solidity
File: /src/vault/VaultController.sol
53: constructor(
    address _owner,
    IAdminProxy _adminProxy,
    IDeploymentController _deploymentController,
    IVaultRegistry _vaultRegistry,
    IPermissionRegistry _permissionRegistry,
    IMultiRewardEscrow _escrow
  ) Owned(_owner) {

File: /src/vault/VaultRegistry.sol
21: constructor(address _owner) Owned(_owner) {}

File: /src/vault/TemplateRegistry.sol
24: constructor(address _owner) Owned(_owner) {}

File: /src/vault/DeploymentController.sol
35: constructor(
    address _owner,
    ICloneFactory _cloneFactory,
    ICloneRegistry _cloneRegistry,
    ITemplateRegistry _templateRegistry
  ) Owned(_owner) {

File: /src/utils/MultiRewardEscrow.sol
30: constructor(address _owner, address _feeRecipient) Owned(_owner) 
```

### [G-11] usage of `uints/ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html) Use a larger size then downcast where needed

*There are 3 instances of this issue:*

**Affected source code:**

```solidity
File: /src/vault/adapter/abstracts/AdapterBase.sol
38: **uint8 internal _decimals;

File: /src/vault/Vault.sol
38: uint8 internal _decimals;

File: /src/utils/MultiRewardStaking.sol
33: uint8 private _decimals;**
```