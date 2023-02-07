## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | Remove or replace unused state variables | 1 |  - |
| [G&#x2011;02] | Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, where appropriate | 3 |  - |
| [G&#x2011;03] | State variables only set in the constructor should be declared `immutable` | 8 |  16776 |
| [G&#x2011;04] | State variables can be packed into fewer storage slots | 1 |  - |
| [G&#x2011;05] | State variables should be cached in stack variables rather than re-reading them from storage | 21 |  2037 |
| [G&#x2011;06] | Multiple accesses of a mapping/array should use a local variable cache | 5 |  210 |
| [G&#x2011;07] | The result of function calls should be cached rather than re-calling the function | 1 |  - |
| [G&#x2011;08] | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables | 2 |  226 |
| [G&#x2011;09] | `internal` functions only called once can be inlined to save gas | 13 |  260 |
| [G&#x2011;10] | Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if`-statement | 2 |  170 |
| [G&#x2011;11] | `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops | 21 |  1260 |
| [G&#x2011;12] | `keccak256()` should only need to be called on a specific string literal once | 9 |  378 |
| [G&#x2011;13] | Optimize names to save gas | 26 |  572 |
| [G&#x2011;14] | Remove unused local variable | 1 |  - |
| [G&#x2011;15] | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 2 |  - |
| [G&#x2011;16] | Using `private` rather than `public` for constants, saves gas | 4 |  - |
| [G&#x2011;17] | Superfluous event fields | 2 |  - |
| [G&#x2011;18] | Functions guaranteed to revert when called by normal users can be marked `payable` | 9 |  189 |
| [G&#x2011;19] | Constructors can be marked `payable` | 9 |  189 |
| [G&#x2011;20] | Don't use `_msgSender()` if not supporting EIP-2771 | 4 |  64 |

Total: 144 instances over 20 issues with **22331 gas** saved

Gas totals use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.





## Gas Optimizations

### [G&#x2011;01]  Remove or replace unused state variables
Saves a storage slot. If the variable is assigned a non-zero value, saves Gsset (**20000 gas**). If it's assigned a zero value, saves Gsreset (**2900 gas**). If the variable remains unassigned, there is no gas savings unless the variable is `public`, in which case the compiler-generated non-payable getter deployment cost is saved. If the state variable is overriding an interface's public function, mark the variable as `constant` or `immutable` so that it does not use a storage slot

*There is 1 instance of this issue:*

```solidity
File: src/vault/Vault.sol

467:      uint256 public assetsCheckpoint;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L467

### [G&#x2011;02]  Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, where appropriate
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (**20000 gas**) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save **~42 gas per access** due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

*There are 3 instances of this issue:*

```solidity
File: src/utils/MultiRewardEscrow.sol

64      mapping(bytes32 => Escrow) public escrows;
65    
66      // User => Escrows
67      mapping(address => bytes32[]) public userEscrowIds;
68      // User => RewardsToken => Escrows
69:     mapping(address => mapping(IERC20 => bytes32[])) public userEscrowIdsByToken;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L64-L69

```solidity
File: src/utils/MultiRewardStaking.sol

216     mapping(address => mapping(IERC20 => uint256)) public userIndex;
217     // user => rewardToken -> accruedRewards
218:    mapping(address => mapping(IERC20 => uint256)) public accruedRewards;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L216-L218

```solidity
File: src/vault/TemplateRegistry.sol

31      mapping(bytes32 => mapping(bytes32 => Template)) public templates;
32      mapping(bytes32 => bytes32[]) public templateIds;
33      mapping(bytes32 => bool) public templateExists;
34    
35:     mapping(bytes32 => bool) public templateCategoryExists;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L31-L35

### [G&#x2011;03]  State variables only set in the constructor should be declared `immutable`
Avoids a Gsset (**20000 gas**) in the constructor, and replaces the first access in each transaction (Gcoldsload - **2100 gas**) and each access thereafter (Gwarmacces - **100 gas**) with a `PUSH32` (**3 gas**). 

While `string`s are not value types, and therefore cannot be `immutable`/`constant` if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract `abstract` with `virtual` functions for the `string` accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

*There are 8 instances of this issue:*

```solidity
File: src/utils/MultiRewardEscrow.sol

/// @audit feeRecipient (constructor)
31:       feeRecipient = _feeRecipient;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L31

```solidity
File: src/vault/DeploymentController.sol

/// @audit cloneFactory (constructor)
41:       cloneFactory = _cloneFactory;

/// @audit cloneRegistry (constructor)
42:       cloneRegistry = _cloneRegistry;

/// @audit templateRegistry (constructor)
43:       templateRegistry = _templateRegistry;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/DeploymentController.sol#L41

```solidity
File: src/vault/VaultController.sol

/// @audit vaultRegistry (constructor)
62:       vaultRegistry = _vaultRegistry;

/// @audit escrow (constructor)
64:       escrow = _escrow;

/// @audit adminProxy (constructor)
61:       adminProxy = _adminProxy;

/// @audit permissionRegistry (constructor)
63:       permissionRegistry = _permissionRegistry;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L62

### [G&#x2011;04]  State variables can be packed into fewer storage slots
If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (**20000 gas**). Reads of the variables can also be cheaper

*There is 1 instance of this issue:*

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

/// @audit Variable ordering with 11 slots instead of the current 12:
///           uint256(32):underlyingBalance, bytes(32):strategyConfig, uint256(32):lastHarvest, uint256(32):harvestCooldown, uint256(32):performanceFee, uint256(32):highWaterMark, uint256(32):INITIAL_CHAIN_ID, bytes32(32):INITIAL_DOMAIN_SEPARATOR, mapping(32):nonces, address(20):strategy, uint8(1):_decimals, address(20):FEE_RECIPIENT
38:       uint8 internal _decimals;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L38

### [G&#x2011;05]  State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (**100 gas**) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*There are 21 instances of this issue:*

```solidity
File: src/utils/MultiRewardEscrow.sol

/// @audit nonce on line 102
104:      bytes32 id = keccak256(abi.encodePacked(token, account, amount, nonce));

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L104

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

/// @audit strategy on line 440
444:              address(strategy).delegatecall(

/// @audit strategy on line 480
481:          strategy.verifyAdapterCompatibility(strategyConfig);

/// @audit strategy on line 481
/// @audit strategyConfig on line 481
482:          strategy.setUp(strategyConfig);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L444

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

/// @audit yVault on line 45
54:           IERC20(_asset).approve(address(yVault), type(uint256).max);

/// @audit yVault on line 90
95:                   yVault.totalSupply(),

/// @audit yVault on line 110
110:          return IERC20(asset()).balanceOf(address(yVault)) + yVault.totalDebt();

/// @audit yVault on line 115
116:              yVault.lockedProfitDegradation();

/// @audit yVault on line 116
119:              uint256 lockedProfit = yVault.lockedProfit();

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L54

```solidity
File: src/vault/VaultController.sol

/// @audit adminProxy on line 124
126:      (bool success, bytes memory returnData) = adminProxy.execute(

/// @audit adminProxy on line 230
238:      adminProxy.execute(adapter, abi.encodeWithSelector(IAdapter.setPerformanceFee.selector, performanceFee));

/// @audit adminProxy on line 457
459:        (success, returnData) = adminProxy.execute(

/// @audit permissionRegistry on line 682
683:            ? !permissionRegistry.endorsed(token)

/// @audit permissionRegistry on line 683
684:            : permissionRegistry.rejected(token)

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L126

```solidity
File: src/vault/Vault.sol

/// @audit highWaterMark on line 448
453:              performanceFee > 0 && shareValue > highWaterMark

/// @audit highWaterMark on line 453
455:                      (shareValue - highWaterMark) * totalSupply(),

/// @audit adapter on line 286
286:          return adapter.convertToAssets(adapter.balanceOf(address(this)));

/// @audit adapter on line 598
599:              adapter.balanceOf(address(this)),

/// @audit adapter on line 608
610:          asset.approve(address(adapter), type(uint256).max);

/// @audit adapter on line 610
612:          adapter.deposit(asset.balanceOf(address(this)), address(this));

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L453

### [G&#x2011;06]  Multiple accesses of a mapping/array should use a local variable cache
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local `storage` or `calldata` variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves **~42 gas per access** due to not having to recalculate the key's keccak256 hash (Gkeccak256 - **30 gas**) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata

*There are 5 instances of this issue:*

```solidity
File: src/utils/MultiRewardEscrow.sol

/// @audit escrows[escrowId] on line 141
141:      return escrows[escrowId].lastUpdateTime != 0 && escrows[escrowId].balance > 0;

/// @audit escrows[escrowId] on line 162
163:        escrows[escrowId].lastUpdateTime = block.timestamp.safeCastTo32();

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L141

```solidity
File: src/utils/MultiRewardStaking.sol

/// @audit rewardInfos[rewardToken] on line 313
314:      rewardInfos[rewardToken].rewardsEndTimestamp = rewardsEndTimestamp;

/// @audit rewardInfos[rewardToken] on line 343
346:      rewardInfos[rewardToken].lastUpdatedTimestamp = block.timestamp.safeCastTo32();

/// @audit rewardInfos[_rewardToken] on line 408
409:      rewardInfos[_rewardToken].lastUpdatedTimestamp = block.timestamp.safeCastTo32();

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L314

### [G&#x2011;07]  The result of function calls should be cached rather than re-calling the function
The instances below point to the second+ call of the function within a single function

*There is 1 instance of this issue:*

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

/// @audit yVault.totalSupply() on line 90
95:                   yVault.totalSupply(),

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L95

### [G&#x2011;08]  `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**

*There are 2 instances of this issue:*

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

158:          underlyingBalance += _underlyingBalance() - underlyingBalance_;

225:              underlyingBalance -= underlyingBalance_ - _underlyingBalance();

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L158

### [G&#x2011;09]  `internal` functions only called once can be inlined to save gas
Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls.

*There are 13 instances of this issue:*

```solidity
File: src/utils/MultiRewardStaking.sol

191     function _lockToken(
192       address user,
193       IERC20 rewardToken,
194       uint256 rewardAmount,
195:      EscrowInfo memory escrowInfo

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L191-L195

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

89:       function _shareValue(uint256 yShares) internal view returns (uint256) {

101:      function _freeFunds() internal view returns (uint256) {

109:      function _yTotalAssets() internal view returns (uint256) {

114:      function _calculateLockedProfit() internal view returns (uint256) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L89

```solidity
File: src/vault/VaultController.sol

120     function _deployVault(VaultInitParams memory vaultData, IDeploymentController _deploymentController)
121       internal
122:      returns (address vault)

141     function _registerCreatedVault(
142       address vault,
143       address staking,
144:      VaultMetadata memory metadata

154:    function _handleVaultStakingRewards(address vault, bytes memory rewardsData) internal {

225     function __deployAdapter(
226       DeploymentArgs memory adapterData,
227       bytes memory baseAdapterData,
228       IDeploymentController _deploymentController
229:    ) internal returns (address adapter) {

242     function _encodeAdapterData(DeploymentArgs memory adapterData, bytes memory baseAdapterData)
243       internal
244:      returns (bytes memory)

256     function _deployStrategy(DeploymentArgs memory strategyData, IDeploymentController _deploymentController)
257       internal
258:      returns (address strategy)

390:    function _registerVault(address vault, VaultMetadata memory metadata) internal {

692:    function _verifyAdapterConfiguration(address adapter, bytes32 adapterId) internal view {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L120-L122

### [G&#x2011;10]  Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if`-statement
`require(a <= b); x = b - a` => `require(a <= b); unchecked { x = b - a }`

*There are 2 instances of this issue:*

```solidity
File: src/utils/MultiRewardStaking.sol

/// @audit if-condition on line 356
357:        amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);

/// @audit if-condition on line 394
395:        elapsed = rewards.rewardsEndTimestamp - rewards.lastUpdatedTimestamp;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L357

### [G&#x2011;11]  `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops
The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**

*There are 21 instances of this issue:*

```solidity
File: src/utils/MultiRewardEscrow.sol

53:       for (uint256 i = 0; i < escrowIds.length; i++) {

155:      for (uint256 i = 0; i < escrowIds.length; i++) {

210:      for (uint256 i = 0; i < tokens.length; i++) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L53

```solidity
File: src/utils/MultiRewardStaking.sol

171:      for (uint8 i; i < _rewardTokens.length; i++) {

373:      for (uint8 i; i < _rewardTokens.length; i++) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L171

```solidity
File: src/vault/PermissionRegistry.sol

42:       for (uint256 i = 0; i < len; i++) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L42

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
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L319

### [G&#x2011;12]  `keccak256()` should only need to be called on a specific string literal once
It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to `bytes4` should also only be done once

*There are 9 instances of this issue:*

```solidity
File: src/utils/MultiRewardStaking.sol

466:                  keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),

495:            keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),

497:            keccak256("1"),

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L466

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

653                                   keccak256(
654                                       "Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"
655:                                  ),

688                       keccak256(
689                           "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"
690:                      ),

692:                      keccak256("1"),

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L653-L655

```solidity
File: src/vault/Vault.sol

685                                   keccak256(
686                                       "Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"
687:                                  ),

720                       keccak256(
721                           "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"
722:                      ),

724:                      keccak256("1"),

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L685-L687

### [G&#x2011;13]  Optimize names to save gas
`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save **128 gas** each during deployment, and renaming functions to have lower method IDs will save **22 gas** per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

*There are 26 instances of this issue:*

```solidity
File: src/interfaces/IMultiRewardEscrow.sol

/// @audit lock(), setFees(), fees()
31:   interface IMultiRewardEscrow {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/IMultiRewardEscrow.sol#L31

```solidity
File: src/interfaces/IMultiRewardStaking.sol

/// @audit addRewardToken(), changeRewardSpeed(), fundReward(), initialize(), rewardInfos(), escrowInfos()
37:   interface IMultiRewardStaking is IERC4626, IOwned, IPermit, IPausable {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/IMultiRewardStaking.sol#L37

```solidity
File: src/interfaces/vault/IAdapter.sol

/// @audit strategy(), strategyConfig(), strategyDeposit(), strategyWithdraw(), setPerformanceFee(), performanceFee(), highWaterMark(), harvest(), harvestCooldown(), setHarvestCooldown(), initialize()
11:   interface IAdapter is IERC4626, IOwned, IPermit, IPausable {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IAdapter.sol#L11

```solidity
File: src/interfaces/vault/ICloneRegistry.sol

/// @audit cloneExists(), addClone()
8:    interface ICloneRegistry is IOwned {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/ICloneRegistry.sol#L8

```solidity
File: src/interfaces/vault/IDeploymentController.sol

/// @audit templateCategoryExists(), templateExists(), addTemplate(), addTemplateCategory(), toggleTemplateEndorsement(), getTemplate(), nominateNewDependencyOwner(), acceptDependencyOwnership(), cloneFactory(), cloneRegistry(), templateRegistry(), PermissionRegistry(), addClone()
11:   interface IDeploymentController is ICloneFactory, ICloneRegistry {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IDeploymentController.sol#L11

```solidity
File: src/interfaces/vault/IPermissionRegistry.sol

/// @audit setPermissions(), endorsed(), rejected()
13:   interface IPermissionRegistry is IOwned {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IPermissionRegistry.sol#L13

```solidity
File: src/interfaces/vault/IStrategy.sol

/// @audit harvest(), verifyAdapterSelectorCompatibility(), verifyAdapterCompatibility(), setUp()
6:    interface IStrategy {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IStrategy.sol#L6

```solidity
File: src/interfaces/vault/ITemplateRegistry.sol

/// @audit templates(), templateCategoryExists(), templateExists(), getTemplateCategories(), getTemplate(), getTemplateIds(), addTemplate(), addTemplateCategory(), toggleTemplateEndorsement()
23:   interface ITemplateRegistry is IOwned {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/ITemplateRegistry.sol#L23

```solidity
File: src/interfaces/vault/IVaultController.sol

/// @audit deployVault(), deployAdapter(), deployStaking(), proposeVaultAdapters(), changeVaultAdapters(), proposeVaultFees(), changeVaultFees(), registerVaults(), addClones(), toggleEndorsements(), toggleRejections(), addStakingRewardsTokens(), changeStakingRewardsSpeeds(), fundStakingRewards(), setEscrowTokenFees(), addTemplateCategories(), toggleTemplateEndorsements(), pauseAdapters(), pauseVaults(), unpauseAdapters(), unpauseVaults(), nominateNewAdminProxyOwner(), acceptAdminProxyOwnership(), setPerformanceFee(), setAdapterPerformanceFees(), performanceFee(), setHarvestCooldown(), setAdapterHarvestCooldowns(), harvestCooldown(), setDeploymentController(), setActiveTemplateId(), activeTemplateId()
19:   interface IVaultController {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IVaultController.sol#L19

```solidity
File: src/interfaces/vault/IVaultRegistry.sol

/// @audit getVault(), getSubmitter(), registerVault()
24:   interface IVaultRegistry is IOwned {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IVaultRegistry.sol#L24

```solidity
File: src/interfaces/vault/IVault.sol

/// @audit accruedManagementFee(), accruedPerformanceFee(), highWaterMark(), assetsCheckpoint(), feesUpdatedAt(), feeRecipient(), deposit(), takeManagementAndPerformanceFees(), adapter(), proposedAdapter(), proposedAdapterTime(), proposeAdapter(), changeAdapter(), fees(), proposedFees(), proposedFeeTime(), proposeFees(), changeFees(), setFeeRecipient(), quitPeriod(), setQuitPeriod(), initialize()
29:   interface IVault is IERC4626 {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IVault.sol#L29

```solidity
File: src/interfaces/vault/IWithRewards.sol

/// @audit claim(), rewardTokens()
6:    interface IWithRewards {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IWithRewards.sol#L6

```solidity
File: src/utils/MultiRewardEscrow.sol

/// @audit getEscrowIdsByUser(), getEscrowIdsByUserAndToken(), getEscrows(), lock(), isClaimable(), getClaimableAmount(), claimRewards(), setFees()
21:   contract MultiRewardEscrow is Owned {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L21

```solidity
File: src/utils/MultiRewardStaking.sol

/// @audit initialize(), deposit(), claimRewards(), addRewardToken(), changeRewardSpeed(), fundReward(), getAllRewardsTokens()
26:   contract MultiRewardStaking is ERC4626Upgradeable, OwnedUpgradeable {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L26

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

/// @audit convertToUnderlyingShares(), harvest(), strategyDeposit(), strategyWithdraw(), setHarvestCooldown(), accruedPerformanceFee(), setPerformanceFee()
27:   abstract contract AdapterBase is

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L27

```solidity
File: src/vault/adapter/abstracts/WithRewards.sol

/// @audit rewardTokens(), claim()
12:   contract WithRewards is EIP165, OnlyStrategy {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/WithRewards.sol#L12

```solidity
File: src/vault/adapter/beefy/IBeefy.sol

/// @audit want(), deposit(), withdrawAll(), balance(), earn(), getPricePerFullShare(), strategy()
6:    interface IBeefyVault {

/// @audit earned(), stakedToken(), rewardToken(), periodFinish(), rewardPerToken(), stake(), exit(), getReward()
29:   interface IBeefyBooster {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/IBeefy.sol#L6

```solidity
File: src/vault/adapter/yearn/IYearn.sol

/// @audit deposit(), pricePerShare(), depositLimit(), token(), lastReport(), lockedProfit(), lockedProfitDegradation(), totalDebt()
8:    interface VaultAPI is IERC20 {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/IYearn.sol#L8

```solidity
File: src/vault/CloneRegistry.sol

/// @audit addClone(), getClonesByCategoryAndId(), getAllClones()
16:   contract CloneRegistry is Owned {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneRegistry.sol#L16

```solidity
File: src/vault/DeploymentController.sol

/// @audit addTemplateCategory(), addTemplate(), toggleTemplateEndorsement(), deploy(), nominateNewDependencyOwner(), acceptDependencyOwnership()
18:   contract DeploymentController is Owned {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/DeploymentController.sol#L18

```solidity
File: src/vault/PermissionRegistry.sol

/// @audit setPermissions(), endorsed(), rejected()
14:   contract PermissionRegistry is Owned {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L14

```solidity
File: src/vault/TemplateRegistry.sol

/// @audit addTemplateCategory(), addTemplate(), toggleTemplateEndorsement(), getTemplateCategories(), getTemplateIds(), getTemplate()
18:   contract TemplateRegistry is Owned {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L18

```solidity
File: src/vault/VaultController.sol

/// @audit deployVault(), deployAdapter(), deployStaking(), proposeVaultAdapters(), changeVaultAdapters(), proposeVaultFees(), changeVaultFees(), setPermissions(), addStakingRewardsTokens(), changeStakingRewardsSpeeds(), fundStakingRewards(), setEscrowTokenFees(), addTemplateCategories(), toggleTemplateEndorsements(), pauseAdapters(), pauseVaults(), unpauseAdapters(), unpauseVaults(), nominateNewAdminProxyOwner(), acceptAdminProxyOwnership(), setPerformanceFee(), setAdapterPerformanceFees(), setHarvestCooldown(), setAdapterHarvestCooldowns(), setDeploymentController(), setActiveTemplateId()
29:   contract VaultController is Owned {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L29

```solidity
File: src/vault/VaultRegistry.sol

/// @audit registerVault(), getVault(), getVaultsByAsset(), getTotalVaults(), getRegisteredAddresses(), getSubmitter()
15:   contract VaultRegistry is Owned {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L15

```solidity
File: src/vault/Vault.sol

/// @audit initialize(), deposit(), accruedManagementFee(), accruedPerformanceFee(), takeManagementAndPerformanceFees(), proposeFees(), changeFees(), setFeeRecipient(), proposeAdapter(), changeAdapter(), setQuitPeriod()
26:   contract Vault is

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L26

### [G&#x2011;14]  Remove unused local variable

*There is 1 instance of this issue:*

```solidity
File: src/vault/Vault.sol

/// @audit highWaterMark_
448:          uint256 highWaterMark_ = highWaterMark;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L448

### [G&#x2011;15]  Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead
> When using elements that are smaller than 32 bytes, your contractâ€™s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra [**22-28 gas**](https://gist.github.com/IllIllI000/9388d20c70f9a4632eb3ca7836f54977) (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

*There are 2 instances of this issue:*

```solidity
File: src/utils/MultiRewardStaking.sol

/// @audit uint32 rewardsEndTimestamp
342:        rewardsEndTimestamp = _calcRewardsEnd(rewards.rewardsEndTimestamp, rewards.rewardsPerSecond, amount);

/// @audit uint224 deltaIndex
406:        deltaIndex = accrued.mulDiv(uint256(10**decimals()), supplyTokens, Math.Rounding.Down).safeCastTo224();

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L342

### [G&#x2011;16]  Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*There are 4 instances of this issue:*

```solidity
File: src/vault/VaultController.sol

36:     bytes32 public immutable VAULT = "Vault";

37:     bytes32 public immutable ADAPTER = "Adapter";

38:     bytes32 public immutable STRATEGY = "Strategy";

39:     bytes32 public immutable STAKING = "Staking";

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L36

### [G&#x2011;17]  Superfluous event fields
`block.timestamp` and `block.number` are added to event information by default so adding them manually wastes gas

*There are 2 instances of this issue:*

```solidity
File: src/vault/Vault.sol

512:      event NewFeesProposed(VaultFees newFees, uint256 timestamp);

569:      event NewAdapterProposed(IERC4626 newAdapter, uint256 timestamp);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L512

### [G&#x2011;18]  Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are 
`CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost

*There are 9 instances of this issue:*

```solidity
File: src/utils/MultiRewardStaking.sol

243     function addRewardToken(
244       IERC20 rewardToken,
245       uint160 rewardsPerSecond,
246       uint256 amount,
247       bool useEscrow,
248       uint192 escrowPercentage,
249       uint32 escrowDuration,
250       uint32 offset
251:    ) external onlyOwner {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L243-L251

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

55        function __AdapterBase_init(bytes memory popERC4626InitData)
56            internal
57:           onlyInitializing

456       function strategyDeposit(uint256 amount, uint256 shares)
457           public
458:          onlyStrategy

467       function strategyWithdraw(uint256 amount, uint256 shares)
468           public
469:          onlyStrategy

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L55-L57

```solidity
File: src/vault/AdminProxy.sol

19      function execute(address target, bytes calldata callData)
20        external
21        onlyOwner
22:       returns (bool success, bytes memory returndata)

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/AdminProxy.sol#L19-L22

```solidity
File: src/vault/CloneRegistry.sol

41      function addClone(
42        bytes32 templateCategory,
43        bytes32 templateId,
44        address clone
45:     ) external onlyOwner {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneRegistry.sol#L41-L45

```solidity
File: src/vault/DeploymentController.sol

99      function deploy(
100       bytes32 templateCategory,
101       bytes32 templateId,
102       bytes calldata data
103:    ) external onlyOwner returns (address clone) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/DeploymentController.sol#L99-L103

```solidity
File: src/vault/TemplateRegistry.sol

67      function addTemplate(
68        bytes32 templateCategory,
69        bytes32 templateId,
70        Template memory template
71:     ) external onlyOwner {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L67-L71

```solidity
File: src/vault/VaultController.sol

579     function toggleTemplateEndorsements(bytes32[] calldata templateCategories, bytes32[] calldata templateIds)
580       external
581:      onlyOwner

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L579-L581

### [G&#x2011;19]  Constructors can be marked `payable`
Payable functions cost less gas to execute, since the compiler does not have to add extra checks to ensure that a payment wasn't provided. A constructor can safely be marked as payable, since only the deployer would be able to pass funds, and the project itself would not pass any funds.

*There are 9 instances of this issue:*

```solidity
File: src/utils/MultiRewardEscrow.sol

30:     constructor(address _owner, address _feeRecipient) Owned(_owner) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L30

```solidity
File: src/vault/AdminProxy.sol

16:     constructor(address _owner) Owned(_owner) {}

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/AdminProxy.sol#L16

```solidity
File: src/vault/CloneFactory.sol

23:     constructor(address _owner) Owned(_owner) {}

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneFactory.sol#L23

```solidity
File: src/vault/CloneRegistry.sol

22:     constructor(address _owner) Owned(_owner) {}

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneRegistry.sol#L22

```solidity
File: src/vault/DeploymentController.sol

35      constructor(
36        address _owner,
37        ICloneFactory _cloneFactory,
38        ICloneRegistry _cloneRegistry,
39        ITemplateRegistry _templateRegistry
40:     ) Owned(_owner) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/DeploymentController.sol#L35-L40

```solidity
File: src/vault/PermissionRegistry.sol

20:     constructor(address _owner) Owned(_owner) {}

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L20

```solidity
File: src/vault/TemplateRegistry.sol

24:     constructor(address _owner) Owned(_owner) {}

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L24

```solidity
File: src/vault/VaultController.sol

53      constructor(
54        address _owner,
55        IAdminProxy _adminProxy,
56        IDeploymentController _deploymentController,
57        IVaultRegistry _vaultRegistry,
58        IPermissionRegistry _permissionRegistry,
59        IMultiRewardEscrow _escrow
60:     ) Owned(_owner) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L53-L60

```solidity
File: src/vault/VaultRegistry.sol

21:     constructor(address _owner) Owned(_owner) {}

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L21

### [G&#x2011;20]  Don't use `_msgSender()` if not supporting EIP-2771
Use `msg.sender` if the code does not implement [EIP-2771 trusted forwarder](https://eips.ethereum.org/EIPS/eip-2771) support

*There are 4 instances of this issue:*

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

119:          _deposit(_msgSender(), receiver, assets, shares);

138:          _deposit(_msgSender(), receiver, assets, shares);

182:          _withdraw(_msgSender(), receiver, owner, assets, shares);

201:          _withdraw(_msgSender(), receiver, owner, assets, shares);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L119


___

## Excluded findings
These findings are excluded from awards calculations because there are publicly-available automated tools that find them. The valid ones appear here for completeness

## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | Using `calldata` instead of `memory` for read-only arguments in `external` functions saves gas | 16 |  1920 |
| [G&#x2011;02] | State variables should be cached in stack variables rather than re-reading them from storage | 6 |  - |
| [G&#x2011;03] | `<array>.length` should not be looked up in every loop of a `for`-loop | 5 |  15 |
| [G&#x2011;04] | Using `bool`s for storage incurs overhead | 3 |  51300 |
| [G&#x2011;05] | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 22 |  110 |
| [G&#x2011;06] | Using `private` rather than `public` for constants, saves gas | 1 |  - |
| [G&#x2011;07] | Functions guaranteed to revert when called by normal users can be marked `payable` | 32 |  672 |

Total: 85 instances over 7 issues with **54017 gas** saved

Gas totals use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.





## Gas Optimizations

### [G&#x2011;01]  Using `calldata` instead of `memory` for read-only arguments in `external` functions saves gas
When a function with a `memory` array is called externally, the `abi.decode()` step has to use a for-loop to copy each index of the `calldata` to the `memory` index. **Each iteration of this for-loop costs at least 60 gas** (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having `memory` arguments, it's still valid for implementation contracs to use `calldata` arguments instead. 

If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it's still more gass-efficient to use `calldata` when the `external` function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I've also flagged instances where the function is `public` but can be marked as `external` since it's not called by the contract, and cases where a constructor is involved

*There are 16 instances of this issue:*

```solidity
File: src/utils/MultiRewardEscrow.sol

/// @audit escrowIds - (valid but excluded finding)
154:    function claimRewards(bytes32[] memory escrowIds) external {

/// @audit tokens - (valid but excluded finding)
/// @audit tokenFees - (valid but excluded finding)
207:    function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L154

```solidity
File: src/utils/MultiRewardStaking.sol

/// @audit _rewardTokens - (valid but excluded finding)
170:    function claimRewards(address user, IERC20[] memory _rewardTokens) external accrueRewards(msg.sender, user) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L170

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

/// @audit adapterInitData - (valid but excluded finding)
/// @audit beefyInitData - (valid but excluded finding)
46        function initialize(
47            bytes memory adapterInitData,
48            address registry,
49            bytes memory beefyInitData
50:       ) external initializer {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L46-L50

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

/// @audit adapterInitData - (valid but excluded finding)
/// @audit (valid but excluded finding)
34        function initialize(
35            bytes memory adapterInitData,
36            address externalRegistry,
37            bytes memory
38:       ) external initializer {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L34-L38

```solidity
File: src/vault/TemplateRegistry.sol

/// @audit template - (valid but excluded finding)
67      function addTemplate(
68        bytes32 templateCategory,
69        bytes32 templateId,
70        Template memory template
71:     ) external onlyOwner {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L67-L71

```solidity
File: src/vault/VaultController.sol

/// @audit vaultData - (valid but excluded finding)
/// @audit adapterData - (valid but excluded finding)
/// @audit strategyData - (valid but excluded finding)
/// @audit rewardsData - (valid but excluded finding)
/// @audit metadata - (valid but excluded finding)
89      function deployVault(
90        VaultInitParams memory vaultData,
91        DeploymentArgs memory adapterData,
92        DeploymentArgs memory strategyData,
93        address staking,
94        bytes memory rewardsData,
95        VaultMetadata memory metadata,
96        uint256 initialDeposit
97:     ) external canCreate returns (address vault) {

/// @audit adapterData - (valid but excluded finding)
/// @audit strategyData - (valid but excluded finding)
186     function deployAdapter(
187       IERC20 asset,
188       DeploymentArgs memory adapterData,
189       DeploymentArgs memory strategyData,
190       uint256 initialDeposit
191:    ) external canCreate returns (address adapter) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L89-L97

### [G&#x2011;02]  State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (**100 gas**) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*There are 6 instances of this issue:*

```solidity
File: src/vault/VaultController.sol

/// @audit adminProxy on line 288 - (valid but excluded finding)
294:          abi.encodeWithSelector(IMultiRewardStaking.initialize.selector, asset, escrow, adminProxy)

/// @audit adminProxy on line 450 - (valid but excluded finding)
457:        IERC20(rewardsToken).transferFrom(msg.sender, address(adminProxy), amount);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L294

```solidity
File: src/vault/Vault.sol

/// @audit proposedFees on line 544 - (valid but excluded finding)
545:          fees = proposedFees;

/// @audit adapter on line 599 - (valid but excluded finding)
604:          asset.approve(address(adapter), 0);

/// @audit adapter on line 604 - (valid but excluded finding)
606:          emit ChangedAdapter(adapter, proposedAdapter);

/// @audit proposedAdapter on line 606 - (valid but excluded finding)
608:          adapter = proposedAdapter;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L545

### [G&#x2011;03]  `<array>.length` should not be looked up in every loop of a `for`-loop
The overheads outlined below are _PER LOOP_, excluding the first loop
* storage arrays incur a Gwarmaccess (**100 gas**)
* memory arrays use `MLOAD` (**3 gas**)
* calldata arrays use `CALLDATALOAD` (**3 gas**)

Caching the length changes each of these to a `DUP<N>` (**3 gas**), and gets rid of the extra `DUP<N>` needed to store the stack offset

*There are 5 instances of this issue:*

```solidity
File: src/utils/MultiRewardEscrow.sol

/// @audit (valid but excluded finding)
53:       for (uint256 i = 0; i < escrowIds.length; i++) {

/// @audit (valid but excluded finding)
155:      for (uint256 i = 0; i < escrowIds.length; i++) {

/// @audit (valid but excluded finding)
210:      for (uint256 i = 0; i < tokens.length; i++) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L53

```solidity
File: src/utils/MultiRewardStaking.sol

/// @audit (valid but excluded finding)
171:      for (uint8 i; i < _rewardTokens.length; i++) {

/// @audit (valid but excluded finding)
373:      for (uint8 i; i < _rewardTokens.length; i++) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L171

### [G&#x2011;04]  Using `bool`s for storage incurs overhead
```solidity
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27
Use `uint256(1)` and `uint256(2)` for true/false to avoid a Gwarmaccess (**[100 gas](https://gist.github.com/IllIllI000/1b70014db712f8572a72378321250058)**) for the extra SLOAD, and to avoid Gsset (**20000 gas**) when changing from `false` to `true`, after having been `true` in the past

*There are 3 instances of this issue:*

```solidity
File: src/vault/CloneRegistry.sol

/// @audit (valid but excluded finding)
28:     mapping(address => bool) public cloneExists;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneRegistry.sol#L28

```solidity
File: src/vault/TemplateRegistry.sol

/// @audit (valid but excluded finding)
33:     mapping(bytes32 => bool) public templateExists;

/// @audit (valid but excluded finding)
35:     mapping(bytes32 => bool) public templateCategoryExists;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L33

### [G&#x2011;05]  `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
Saves **5 gas per loop**

*There are 22 instances of this issue:*

```solidity
File: src/utils/MultiRewardEscrow.sol

/// @audit (valid but excluded finding)
53:       for (uint256 i = 0; i < escrowIds.length; i++) {

/// @audit (valid but excluded finding)
102:      nonce++;

/// @audit (valid but excluded finding)
155:      for (uint256 i = 0; i < escrowIds.length; i++) {

/// @audit (valid but excluded finding)
210:      for (uint256 i = 0; i < tokens.length; i++) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L53

```solidity
File: src/utils/MultiRewardStaking.sol

/// @audit (valid but excluded finding)
171:      for (uint8 i; i < _rewardTokens.length; i++) {

/// @audit (valid but excluded finding)
373:      for (uint8 i; i < _rewardTokens.length; i++) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L171

```solidity
File: src/vault/PermissionRegistry.sol

/// @audit (valid but excluded finding)
42:       for (uint256 i = 0; i < len; i++) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L42

```solidity
File: src/vault/VaultController.sol

/// @audit (valid but excluded finding)
319:      for (uint8 i = 0; i < len; i++) {

/// @audit (valid but excluded finding)
337:      for (uint8 i = 0; i < len; i++) {

/// @audit (valid but excluded finding)
357:      for (uint8 i = 0; i < len; i++) {

/// @audit (valid but excluded finding)
374:      for (uint8 i = 0; i < len; i++) {

/// @audit (valid but excluded finding)
437:      for (uint256 i = 0; i < len; i++) {

/// @audit (valid but excluded finding)
494:      for (uint256 i = 0; i < len; i++) {

/// @audit (valid but excluded finding)
523:      for (uint256 i = 0; i < len; i++) {

/// @audit (valid but excluded finding)
564:      for (uint256 i = 0; i < len; i++) {

/// @audit (valid but excluded finding)
587:      for (uint256 i = 0; i < len; i++) {

/// @audit (valid but excluded finding)
607:      for (uint256 i = 0; i < len; i++) {

/// @audit (valid but excluded finding)
620:      for (uint256 i = 0; i < len; i++) {

/// @audit (valid but excluded finding)
633:      for (uint256 i = 0; i < len; i++) {

/// @audit (valid but excluded finding)
646:      for (uint256 i = 0; i < len; i++) {

/// @audit (valid but excluded finding)
766:      for (uint256 i = 0; i < len; i++) {

/// @audit (valid but excluded finding)
806:      for (uint256 i = 0; i < len; i++) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L319

### [G&#x2011;06]  Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*There is 1 instance of this issue:*

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

/// @audit (valid but excluded finding)
31:       uint256 public constant BPS_DENOMINATOR = 10_000;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L31

### [G&#x2011;07]  Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are 
`CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost

*There are 32 instances of this issue:*

```solidity
File: src/utils/MultiRewardEscrow.sol

/// @audit (valid but excluded finding)
207:    function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L207

```solidity
File: src/utils/MultiRewardStaking.sol

/// @audit (valid but excluded finding)
296:    function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L296

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

/// @audit (valid but excluded finding)
500:      function setHarvestCooldown(uint256 newCooldown) external onlyOwner {

/// @audit (valid but excluded finding)
549:      function setPerformanceFee(uint256 newFee) public onlyOwner {

/// @audit (valid but excluded finding)
574:      function pause() external onlyOwner {

/// @audit (valid but excluded finding)
582:      function unpause() external onlyOwner {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L500

```solidity
File: src/vault/adapter/abstracts/WithRewards.sol

/// @audit (valid but excluded finding)
15:     function claim() public virtual onlyStrategy {}

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/WithRewards.sol#L15

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

/// @audit (valid but excluded finding)
221:      function claim() public override onlyStrategy {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L221

```solidity
File: src/vault/CloneFactory.sol

/// @audit (valid but excluded finding)
39:     function deploy(Template calldata template, bytes calldata data) external onlyOwner returns (address clone) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneFactory.sol#L39

```solidity
File: src/vault/DeploymentController.sol

/// @audit (valid but excluded finding)
55:     function addTemplateCategory(bytes32 templateCategory) external onlyOwner {

/// @audit (valid but excluded finding)
80:     function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

/// @audit (valid but excluded finding)
121:    function nominateNewDependencyOwner(address _owner) external onlyOwner {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/DeploymentController.sol#L55

```solidity
File: src/vault/PermissionRegistry.sol

/// @audit (valid but excluded finding)
38:     function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L38

```solidity
File: src/vault/TemplateRegistry.sol

/// @audit (valid but excluded finding)
52:     function addTemplateCategory(bytes32 templateCategory) external onlyOwner {

/// @audit (valid but excluded finding)
102:    function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L52

```solidity
File: src/vault/VaultController.sol

/// @audit (valid but excluded finding)
408:    function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

/// @audit (valid but excluded finding)
543:    function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {

/// @audit (valid but excluded finding)
561:    function addTemplateCategories(bytes32[] calldata templateCategories) external onlyOwner {

/// @audit (valid but excluded finding)
723:    function nominateNewAdminProxyOwner(address newOwner) external onlyOwner {

/// @audit (valid but excluded finding)
751:    function setPerformanceFee(uint256 newFee) external onlyOwner {

/// @audit (valid but excluded finding)
764:    function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {

/// @audit (valid but excluded finding)
791:    function setHarvestCooldown(uint256 newCooldown) external onlyOwner {

/// @audit (valid but excluded finding)
804:    function setAdapterHarvestCooldowns(address[] calldata adapters) external onlyOwner {

/// @audit (valid but excluded finding)
832:    function setDeploymentController(IDeploymentController _deploymentController) external onlyOwner {

/// @audit (valid but excluded finding)
864:    function setActiveTemplateId(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L408

```solidity
File: src/vault/VaultRegistry.sol

/// @audit (valid but excluded finding)
44:     function registerVault(VaultMetadata calldata _metadata) external onlyOwner {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L44

```solidity
File: src/vault/Vault.sol

/// @audit (valid but excluded finding)
525:      function proposeFees(VaultFees calldata newFees) external onlyOwner {

/// @audit (valid but excluded finding)
553:      function setFeeRecipient(address _feeRecipient) external onlyOwner {

/// @audit (valid but excluded finding)
578:      function proposeAdapter(IERC4626 newAdapter) external onlyOwner {

/// @audit (valid but excluded finding)
629:      function setQuitPeriod(uint256 _quitPeriod) external onlyOwner {

/// @audit (valid but excluded finding)
643:      function pause() external onlyOwner {

/// @audit (valid but excluded finding)
648:      function unpause() external onlyOwner {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L525
