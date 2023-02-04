## Summary<a name="Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | `abi.encode()` is less efficient than `abi.encodepacked()` | 7 | 700 |
| [GAS&#x2011;2](#GAS&#x2011;2) | State variables only set in the `constructor` should be declared immutable | 8 | 16800 |
| [GAS&#x2011;3](#GAS&#x2011;3) | Setting the `constructor` to `payable` | 9 | 117 |
| [GAS&#x2011;4](#GAS&#x2011;4) | Do not calculate constants | 1 | - |
| [GAS&#x2011;5](#GAS&#x2011;5) | Empty Blocks Should Be Removed Or Emit Something | 9 | - |
| [GAS&#x2011;6](#GAS&#x2011;6) | Using `delete` statement can save gas | 2 | - |
| [GAS&#x2011;7](#GAS&#x2011;7) | Use `assembly` to write address storage values | 4 | - |
| [GAS&#x2011;8](#GAS&#x2011;8) | Use hardcoded address instead `address(this)` | 26 | - |
| [GAS&#x2011;9](#GAS&#x2011;9) | `internal` functions only called once can be inlined to save gas | 14 | 308 |
| [GAS&#x2011;10](#GAS&#x2011;10) | Multiple Address Mappings Can Be Combined Into A Single Mapping Of An Address To A Struct, Where Appropriate | 7 | - |
| [GAS&#x2011;11](#GAS&#x2011;11) | Optimize names to save gas | 15 | 330 |
| [GAS&#x2011;12](#GAS&#x2011;12) | `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables | 12 | - |
| [GAS&#x2011;13](#GAS&#x2011;13) | Public Functions To External | 34 | - |
| [GAS&#x2011;14](#GAS&#x2011;14) | Help The Optimizer By Saving A Storage Variable’s Reference Instead Of Repeatedly Fetching It | 4 | - |
| [GAS&#x2011;15](#GAS&#x2011;15) | Superfluous event fields | 2 | - |
| [GAS&#x2011;16](#GAS&#x2011;16) | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 153 | - |
| [GAS&#x2011;17](#GAS&#x2011;17) | `++i`/`i++` Should Be `unchecked{++i}`/`unchecked{i++}` When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops | 21 | 735 |
| [GAS&#x2011;18](#GAS&#x2011;18) | Use v4.8.1 OpenZeppelin contracts | 1 | - |
| [GAS&#x2011;19](#GAS&#x2011;19) | Use solidity version 0.8.17 to gain some gas boost | 35 | 3080 |
| [GAS&#x2011;20](#GAS&#x2011;20) | Using `storage` instead of `memory` saves gas | 19 | 6650 |
| [GAS&#x2011;21](#GAS&#x2011;21) | Using `10**X` for constants isn't gas efficient  | 1 | - |

Total: 384 contexts over 21 issues

## Gas Optimizations

### <a href="#Summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> `abi.encode()` is less efficient than `abi.encodepacked()`

See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison 

#### <ins>Proof Of Concept</ins>


```solidity
465: abi.encode(
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L465

```solidity
494: abi.encode(
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L494

```solidity
684: abi.encode(
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L684

```solidity
719: abi.encode(
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L719

```solidity
219: abi.encode(asset, address(adminProxy), IStrategy(strategy), harvestCooldown, requiredSigs, strategyData.data),
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L219

```solidity
652: abi.encode(
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L652

```solidity
687: abi.encode(
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L687







### <a href="#Summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> State variables only set in the `constructor` should be declared immutable

Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmacces (100 gas) with a PUSH32 (3 gas).

#### <ins>Proof Of Concept</ins>

```solidity
31: feeRecipient = _feeRecipient
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L31

```solidity
41: cloneFactory = _cloneFactory
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol#L41

```solidity
42: cloneRegistry = _cloneRegistry
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol#L42

```solidity
43: templateRegistry = _templateRegistry
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol#L43

```solidity
61: adminProxy = _adminProxy
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L61

```solidity
62: vaultRegistry = _vaultRegistry
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L62

```solidity
63: permissionRegistry = _permissionRegistry
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L63

```solidity
64: escrow = _escrow
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L64





### <a href="#Summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Setting the `constructor` to `payable`

Saves ~13 gas per instance

#### <ins>Proof Of Concept</ins>

```solidity
30: constructor(address _owner, address _feeRecipient) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L30

```solidity
16: constructor(address _owner) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/AdminProxy.sol#L16

```solidity
23: constructor(address _owner) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneFactory.sol#L23

```solidity
22: constructor(address _owner) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol#L22

```solidity
35: constructor(
    address _owner,
    ICloneFactory _cloneFactory,
    ICloneRegistry _cloneRegistry,
    ITemplateRegistry _templateRegistry
  ) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol#L35

```solidity
20: constructor(address _owner) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol#L20

```solidity
24: constructor(address _owner) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol#L24

```solidity
53: constructor(
    address _owner,
    IAdminProxy _adminProxy,
    IDeploymentController _deploymentController,
    IVaultRegistry _vaultRegistry,
    IPermissionRegistry _permissionRegistry,
    IMultiRewardEscrow _escrow
  ) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L53

```solidity
21: constructor(address _owner) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol#L21





### <a href="#Summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

#### <ins>Proof Of Concept</ins>

```solidity
25: uint256 constant DEGRADATION_COEFFICIENT = 10**18;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L25










### <a href="#Summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Empty Blocks Should Be Removed Or Emit Something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}})

#### <ins>Proof Of Concept</ins>

```solidity
7: function supportsInterface(bytes4 interfaceId) public view virtual returns (bool) {}
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/EIP165.sol#L7

```solidity
477: {}
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L477

```solidity
258: function _totalAssets() internal view virtual returns (uint256) {}
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L258

```solidity
265: function _underlyingBalance() internal view virtual returns (uint256) {}
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L265

```solidity
258: {}
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L258

```solidity
265: {}
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L265

```solidity
276: {}
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L276

```solidity
13: function rewardTokens() external view virtual returns (address[] memory) {}
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/WithRewards.sol#L13

```solidity
15: function claim() public virtual onlyStrategy {}
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/WithRewards.sol#L15








### <a href="#Summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Using `delete` statement can save gas

#### <ins>Proof Of Concept</ins>

```solidity
186: accruedRewards[user][_rewardTokens[i]] = 0;

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L186

```solidity
577: underlyingBalance = 0;

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L577




### <a href="#Summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Use `assembly` to write address storage values

#### <ins>Proof Of Concept</ins>

```solidity
372: _rewardTokens = rewardTokens;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L372

```solidity
98: _deploymentController = deploymentController;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L98

```solidity
318: _cloneRegistry = cloneRegistry;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L318

```solidity
147: _bestVault = yVault;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L147




### <a href="#Summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry's script.sol and solmate's `LibRlp.sol` contracts can help achieve this.

References: 
https://book.getfoundry.sh/reference/forge-std/compute-create-address 

https://twitter.com/transmissions11/status/1518507047943245824

#### <ins>Proof Of Concept</ins>

```solidity
100: token.safeTransferFrom(msg.sender, address(this), amount);

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L100

```solidity
113: IERC20(asset()).safeTransferFrom(caller, address(this), assets);

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L113

```solidity
259: rewardToken.safeTransferFrom(msg.sender, address(this), amount);

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L259

```solidity
305: uint256 remainder = rewardToken.balanceOf(address(this));

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L305

```solidity
334: rewardToken.safeTransferFrom(msg.sender, address(this), amount);

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L334

```solidity
499: address(this)

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L499

```solidity
153: asset.safeTransferFrom(msg.sender, address(this), assets);
155: adapter.deposit(assets, address(this));

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L153

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L155



```solidity
193: asset.safeTransferFrom(msg.sender, address(this), assets);
195: adapter.deposit(assets, address(this));

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L193

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L195



```solidity
237: adapter.withdraw(assets, receiver, address(this));

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L237

```solidity
275: adapter.withdraw(assets, receiver, address(this));

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L275

```solidity
286: return adapter.convertToAssets(adapter.balanceOf(address(this)));

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L286

```solidity
599: adapter.balanceOf(address(this)),
600: address(this),
599: address(this)
612: adapter.deposit(asset.balanceOf(address(this)), address(this));

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L599

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L600

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L599

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L612



```solidity
726: address(this)

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L726

```solidity
170: asset.safeTransferFrom(msg.sender, address(this), initialDeposit);

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L170

```solidity
526: rewardTokens[i].transferFrom(msg.sender, address(this), amounts[i]);

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L526

```solidity
153: IERC20(asset()).safeTransferFrom(caller, address(this), assets);

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L153

```solidity
250: ? IERC20(asset()).balanceOf(address(this))

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L250

```solidity
694: address(this)

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L694

```solidity
118: return beefyBalanceCheck.balanceOf(address(this));

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L118

```solidity
199: beefyBooster.stake(beefyVault.balanceOf(address(this)));

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L199

```solidity
85: return yVault.balanceOf(address(this));

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L85



#### <ins>Recommended Mitigation Steps</ins>

Use hardcoded `address`







### <a href="#Summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> `internal` functions only called once can be inlined to save gas

#### <ins>Proof Of Concept</ins>


```solidity
137: function _transfer

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L137

```solidity
191: function _lockToken

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L191

```solidity
120: function _deployVault

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L120

```solidity
141: function _registerCreatedVault

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L141

```solidity
154: function _handleVaultStakingRewards

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L154

```solidity
225: function __deployAdapter

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L225

```solidity
242: function _encodeAdapterData

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L242

```solidity
256: function _deployStrategy

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L256

```solidity
390: function _registerVault

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L390

```solidity
692: function _verifyAdapterConfiguration

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L692


```solidity
479: function _verifyAndSetupStrategy

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L479


```solidity
101: function _freeFunds

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L101

```solidity
109: function _yTotalAssets

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L109

```solidity
114: function _calculateLockedProfit

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L114






### <a href="#Summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Multiple Address Mappings Can Be Combined Into A Single Mapping Of An Address To A Struct, Where Appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot.

#### <ins>Proof Of Concept</ins>


```solidity
28: mapping(address => VaultMetadata) public metadata;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol#L28

```solidity
31: mapping(address => address[]) public vaultsByAsset;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol#L31


#### <ins>Recommended Mitigation Steps</ins>

Instead can use:
```solidity
struct structExample{
    VaultMetadata metadata;
    address[] vaultsByAsset;
}



### <a href="#Summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

See more <a href="https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92">here</a>

#### <ins>Proof Of Concept</ins>

All in-scope contracts

#### <ins>Recommended Mitigation Steps</ins>
Find a lower method ID name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas
For example, the function IDs in the Gauge.sol contract will be the most used; A lower method ID may be given.



### <a href="#Summary">[GAS&#x2011;12]</a><a name="GAS&#x2011;12"> `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables

#### <ins>Proof Of Concept</ins>


```solidity
110: amount -= fee;

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L110

```solidity
162: escrows[escrowId].balance -= claimable;

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L162

```solidity
357: amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L357

```solidity
408: rewardInfos[_rewardToken].index += deltaIndex;

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L408

```solidity
431: accruedRewards[_user][_rewardToken] += supplierDelta;

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L431

```solidity
228: shares += feeShares;

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L228

```solidity
343: shares += shares.mulDiv(

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L343

```solidity
365: assets += assets.mulDiv(

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L365

```solidity
387: assets -= assets.mulDiv(

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L387

```solidity
158: underlyingBalance += _underlyingBalance() - underlyingBalance_;

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L158

```solidity
225: underlyingBalance -= underlyingBalance_ - _underlyingBalance();

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L225

```solidity
178: assets -= assets.mulDiv(

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L178




### <a href="#Summary">[GAS&#x2011;13]</a><a name="GAS&#x2011;13"> Public Functions To External

The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

#### <ins>Proof Of Concept</ins>


```solidity
function supportsInterface(bytes4 interfaceId) public view virtual returns (bool) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/EIP165.sol#L7

```solidity
function name() public view override(ERC20Upgradeable, IERC20Metadata) returns (string memory) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L59

```solidity
function symbol() public view override(ERC20Upgradeable, IERC20Metadata) returns (string memory) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L63

```solidity
function decimals() public view override(ERC20Upgradeable, IERC20Metadata) returns (uint8) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L67

```solidity
function permit(
    address owner,
    address spender,
    uint256 value,
    uint256 deadline,
    uint8 v,
    bytes32 r,
    bytes32 s
  ) public virtual {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L445

```solidity
function DOMAIN_SEPARATOR() public view returns (bytes32) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L487

```solidity
function decimals() public view override returns (uint8) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L100

```solidity
function deposit(uint256 assets) public returns (uint256) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L124

```solidity
function withdraw(uint256 assets) public returns (uint256) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L200

```solidity
function withdraw(
        uint256 assets,
        address receiver,
        address owner
    ) public nonReentrant syncFeeCheckpoint returns (uint256 shares) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L200

```solidity
function redeem(
        uint256 shares,
        address receiver,
        address owner
    ) public nonReentrant returns (uint256 assets) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L242

```solidity
function totalAssets() public view returns (uint256) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L285

```solidity
function convertToShares(uint256 assets) public view returns (uint256) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L294

```solidity
function convertToAssets(uint256 shares) public view returns (uint256) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L308

```solidity
function previewMint(uint256 shares) public view returns (uint256 assets) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L340

```solidity
function maxDeposit(address caller) public view returns (uint256) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L399

```solidity
function accruedManagementFee() public view returns (uint256) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L429

```solidity
function accruedPerformanceFee() public view returns (uint256) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L447

```solidity
function permit(
        address owner,
        address spender,
        uint256 value,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) public virtual {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L664

```solidity
function DOMAIN_SEPARATOR() public view returns (bytes32) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L709

```solidity
function addStakingRewardsTokens(address[] memory vaults, bytes[] memory rewardTokenData) public {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L433

```solidity
function withdraw(
        uint256 assets,
        address receiver,
        address owner
    ) public virtual override returns (uint256) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L173

```solidity
function redeem(
        uint256 shares,
        address receiver,
        address owner
    ) public virtual override returns (uint256) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L193

```solidity
function totalAssets() public view override returns (uint256) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L247

```solidity
function maxMint(address) public view virtual override returns (uint256) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L419

```solidity
function harvest() public takeFees {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L438

```solidity
function accruedPerformanceFee() public view returns (uint256) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L529

```solidity
function setPerformanceFee(uint256 newFee) public onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L549

```solidity
function permit(
        address owner,
        address spender,
        uint256 value,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) public virtual {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L632

```solidity
function DOMAIN_SEPARATOR() public view virtual returns (bytes32) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L677

```solidity
function claim() public virtual onlyStrategy {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/WithRewards.sol#L15

```solidity
function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/WithRewards.sol#L21

```solidity
function claim() public override onlyStrategy {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L221

```solidity
function maxDeposit(address) public view override returns (uint256) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L144






### <a href="#Summary">[GAS&#x2011;14]</a><a name="GAS&#x2011;14"> Help The Optimizer By Saving A Storage Variable's Reference Instead Of Repeatedly Fetching It

To help the optimizer, declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array.
The effect can be quite significant.
As an example, instead of repeatedly calling someMap[someIndex], save its reference like this: SomeStruct storage someStruct = someMap[someIndex] and use it.

#### <ins>Proof Of Concept</ins>


```solidity
157: Escrow memory escrow = escrows[escrowId];
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L157


```solidity
172: uint256 rewardAmount = accruedRewards[user][_rewardTokens[i]];
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L172

```solidity
186: accruedRewards[user][_rewardTokens[i]] = 0;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L186

```solidity
375: RewardInfo memory rewards = rewardInfos[rewardToken];
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L375






### <a href="#Summary">[GAS&#x2011;15]</a><a name="GAS&#x2011;15"> Superfluous event fields

`block.number` and `block.timestamp` are added to the event information by default, so adding them manually will waste additional gas.

#### <ins>Proof Of Concept</ins>


```solidity
512: event NewFeesProposed(VaultFees newFees, uint256 timestamp);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L512

```solidity
569: event NewAdapterProposed(IERC4626 newAdapter, uint256 timestamp);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L569






### <a href="#Summary">[GAS&#x2011;16]</a><a name="GAS&#x2011;16"> Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

#### <ins>Proof Of Concept</ins>


```solidity
11: uint32 start;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol#L11

```solidity
13: uint32 end;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol#L13

```solidity
15: uint32 lastUpdateTime;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol#L15

```solidity
16: uint64 ONE;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L16

```solidity
18: uint160 rewardsPerSecond;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L18

```solidity
21: uint32 rewardsEndTimestamp;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L21

```solidity
23: uint224 index;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L23

```solidity
25: uint32 lastUpdatedTimestamp;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L25

```solidity
30: uint192 escrowPercentage;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L30

```solidity
32: uint32 escrowDuration;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L32

```solidity
34: uint32 offset;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L34

```solidity
9: uint64 deposit;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol#L9

```solidity
10: uint64 withdrawal;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol#L10

```solidity
11: uint64 management;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol#L11

```solidity
12: uint64 performance;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol#L12

```solidity
114: uint32 start = block.timestamp.safeCastTo32() + offset;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L114

```solidity
33: uint8 private _decimals;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L33

```solidity
274: uint64 ONE = (10**IERC20Metadata(address(rewardToken)).decimals()).safeCastTo64();
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L274

```solidity
307: uint32 prevEndTime = rewards.rewardsEndTimestamp;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L307

```solidity
340: uint32 rewardsEndTimestamp = rewards.rewardsEndTimestamp;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L340

```solidity
404: uint224 deltaIndex;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L404

```solidity
38: uint8 internal _decimals;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L38

```solidity
645: uint8 len = uint8(vaults.length);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L645

```solidity
583: uint8 len = uint8(templateCategories.length);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L583

```solidity
805: uint8 len = uint8(adapters.length);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L805

```solidity
38: uint8 internal _decimals;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L38

```solidity
238: interfaceId == type(IAdapter).interfaceId;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L238






### <a href="#Summary">[GAS&#x2011;17]</a><a name="GAS&#x2011;17"> `++i`/`i++` Should Be `unchecked{++i}`/`unchecked{i++}` When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

#### <ins>Proof Of Concept</ins>


```solidity
53: for (uint256 i = 0; i < escrowIds.length; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L53

```solidity
155: for (uint256 i = 0; i < escrowIds.length; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L155

```solidity
210: for (uint256 i = 0; i < tokens.length; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L210

```solidity
171: for (uint8 i; i < _rewardTokens.length; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L171

```solidity
373: for (uint8 i; i < _rewardTokens.length; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L373

```solidity
42: for (uint256 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol#L42

```solidity
319: for (uint8 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L319

```solidity
337: for (uint8 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L337

```solidity
357: for (uint8 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L357

```solidity
374: for (uint8 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L374

```solidity
437: for (uint256 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L437

```solidity
494: for (uint256 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L494

```solidity
523: for (uint256 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L523

```solidity
564: for (uint256 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L564

```solidity
587: for (uint256 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L587

```solidity
607: for (uint256 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L607

```solidity
620: for (uint256 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L620

```solidity
633: for (uint256 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L633

```solidity
646: for (uint256 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L646

```solidity
766: for (uint256 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L766

```solidity
806: for (uint256 i = 0; i < len; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L806






### <a href="#Summary">[GAS&#x2011;18]</a><a name="GAS&#x2011;18"> Use v4.8.1 OpenZeppelin contracts

The upcoming v4.8.1 version of OpenZeppelin provides many small gas optimizations.
https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.8.1

#### <ins>Proof Of Concept</ins>

```solidity
    "openzeppelin-upgradeable": "npm:@openzeppelin/contracts-upgradeable@4.7.1"
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/../package.json#L26



#### <ins>Recommended Mitigation Steps</ins>

Update OpenZeppelin Contracts Usage in package.json and require at least version of 4.8.1



### <a href="#Summary">[GAS&#x2011;19]</a><a name="GAS&#x2011;19"> Use solidity version 0.8.17 to gain some gas boost

Upgrade to the latest solidity version 0.8.17 to get additional gas savings.

#### <ins>Proof Of Concept</ins>


```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IEIP165.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IAdapter.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IAdminProxy.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ICloneFactory.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ICloneRegistry.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IDeploymentController.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IERC4626.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IPermissionRegistry.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IStrategy.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ITemplateRegistry.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVaultController.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVaultRegistry.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IWithRewards.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/EIP165.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/AdminProxy.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneFactory.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/OnlyStrategy.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/WithRewards.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/IBeefy.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/IYearn.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L2





### <a href="#Summary">[GAS&#x2011;20]</a><a name="GAS&#x2011;20"> Using `storage` instead of `memory` saves gas

When fetching data from a `storage` location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from `storage`, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new `memory` variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another `memory` array/struct

#### <ins>Proof Of Concept</ins>


```solidity
157: Escrow memory escrow = escrows[escrowId];

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L157

```solidity
176: EscrowInfo memory escrowInfo = escrowInfos[_rewardTokens[i]];

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L176

```solidity
254: RewardInfo memory rewards = rewardInfos[rewardToken];
297: RewardInfo memory rewards = rewardInfos[rewardToken];
328: RewardInfo memory rewards = rewardInfos[rewardToken];
375: RewardInfo memory rewards = rewardInfos[rewardToken];

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L254

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L297

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L328

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L375



```solidity
254: RewardInfo memory rewards = rewardInfos[rewardToken];
297: RewardInfo memory rewards = rewardInfos[rewardToken];
328: RewardInfo memory rewards = rewardInfos[rewardToken];
375: RewardInfo memory rewards = rewardInfos[rewardToken];

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L254

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L297

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L328

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L375



```solidity
254: RewardInfo memory rewards = rewardInfos[rewardToken];
297: RewardInfo memory rewards = rewardInfos[rewardToken];
328: RewardInfo memory rewards = rewardInfos[rewardToken];
375: RewardInfo memory rewards = rewardInfos[rewardToken];

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L254

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L297

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L328

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L375



```solidity
254: RewardInfo memory rewards = rewardInfos[rewardToken];
297: RewardInfo memory rewards = rewardInfos[rewardToken];
328: RewardInfo memory rewards = rewardInfos[rewardToken];
375: RewardInfo memory rewards = rewardInfos[rewardToken];

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L254

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L297

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L328

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L375



```solidity
414: RewardInfo memory rewards = rewardInfos[_rewardToken];

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L414





### <a href="#Summary">[GAS&#x2011;21]</a><a name="GAS&#x2011;21"> Using `10**X` for constants isn't gas efficient 

In Solidity, a constant expression in a variable will compute the expression everytime the variable is called. It's not the result of the expression that is stored, but the expression itself.

As Solidity supports the scientific notation, constants of form 10**X can be rewritten as 1eX to save the gas cost from the calculation with the exponentiation operator **.

#### <ins>Proof Of Concept</ins>


```solidity
25: uint256 constant DEGRADATION_COEFFICIENT = 10**18;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L25



#### <ins>Recommended Mitigation Steps</ins>

Replace 10**X with 1eX
