### Gas Optimizations List
| Number | Optimization Details | Context |
|:--:|:-------| :-----:|
| [G-01] | Gas overflow during iteration (DoS) | 1 |
| [G-02] |Duplicated require()/if() checks should be refactored to a modifier or function| 4 |
| [G-03] |State variables only set in the constructor should be declared immutable |10 |
| [G-04] |Using ``delete``  instead of setting mapping/state variable ``0`` saves gas  | 2 |
| [G-05] | Using `storage` instead of `memory` for `structs/arrays` saves gas |7|
| [G-06] | Add ``unchecked {}`` for subtractions where the operands cannot underflow because of a previous ``require`` or ``if`` statement | 3 |
| [G-07] |Use ``assembly`` to write _address storage values_  | 19 |
| [G-08] | Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead | 4 |
| [G-09] | Don't use _msgSender() if not supporting EIP-2771  | 4 |
| [G-10] |``internal`` functions only called once can be inlined to save gas | 4 |
| [G-11] | Emitting storage values instead of the memory one |1 |
| [G-12] |Superfluous event fields  | 2 |
| [G-13] |Empty blocks should be removed or emit something  | 7 |
| [G-14] |Setting the _constructor_ to `payable` (117 gas) |9 |
| [G-15] |`x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables  | 2 |
| [G-16] |``>=`` costs less gas than ``>`` |4 |
| [G-17] |Use nested if and, avoid multiple check combinations  | 2 |
| [G-18] |Sort Solidity operations using short-circuit mode |5 |
| [G-19] |Optimize names to save gas (22 gas) |All contracts  |
| [G-20] |External visibility can be used in public functions | 13 |
| [G-21] |Use a more recent version of solidity | All contracts |
| [G-22] |Cheaper to calculate domain separator every time | 3 |
| [G-23] |Use hardcode address instead ``address(this)`` | 20 |

Total 23 issues

### [G-01] Gas overflow during iteration (DoS)

Each iteration of the cycle requires a gas flow. A moment may come when more gas is required than it is allocated to record one block. In this case, all iterations of the loop will fail.

1 result - 1 file:
```diff
src/utils/MultiRewardEscrow.sol:
  153     */
  154:   function claimRewards(bytes32[] memory escrowIds) external {
+              require(escrowIds.length< max escrowIdsLengt, "max length");

  155:     for (uint256 i = 0; i < escrowIds.length; i++) {
  156:       bytes32 escrowId = escrowIds[i];
  157:       Escrow memory escrow = escrows[escrowId];
  158: 
  159:       uint256 claimable = _getClaimableAmount(escrow);
  160:       if (claimable == 0) revert NotClaimable(escrowId);
  161: 
  162:       escrows[escrowId].balance -= claimable;
  163:       escrows[escrowId].lastUpdateTime = block.timestamp.safeCastTo32();
  164: 
  165:       escrow.token.safeTransfer(escrow.account, claimable);
  166:       emit RewardsClaimed(escrow.token, escrow.account, claimable);
  167:     }
  168:   }

```

Here are the data available in the covered contracts. The use of this situation in contracts that are not covered will also provide gas optimization.

### [G-02] Duplicated require()/if() checks should be refactored to a modifier or function

4 results - 1 file:
```solidity

src\vault\Vault.sol:

  141:         if (receiver == address(0)) revert InvalidReceiver();

  177:         if (receiver == address(0)) revert InvalidReceiver();

  216:         if (receiver == address(0)) revert InvalidReceiver();
 
  258:         if (receiver == address(0)) revert InvalidReceiver();

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L141


**Recommendation:**

You can consider adding a modifier like below.

```solidity
 modifer checkReceiver () {
        if (receiver == address(0)) revert InvalidReceiver();
        _;
 }
```

Here are the data available in the covered contracts. The use of this situation in contracts that are not covered will also provide gas optimization.

### [G-03] State variables only set in the constructor should be declared immutable

Avoids a Gsset (20000 gas) in the constructor and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

10 result - 6 files:
```solidity
src\utils\MultiRewardEscrow.sol:
   
  30   constructor(address _owner, address _feeRecipient) Owned(_owner) {
  31:     feeRecipient = _feeRecipient;

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L31

```solidity
src\vault\VaultController.sol:

  53   constructor(
  61:     adminProxy = _adminProxy;
  62:     vaultRegistry = _vaultRegistry;
  63:     permissionRegistry = _permissionRegistry;
  64:     escrow = _escrow;

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L61-L64

```solidity
src\utils\MultiRewardStaking.sol:
  41   function initialize(
  53:     escrow = _escrow;

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L53

```solidity
src\vault\Vault.sol:
  57     function initialize(
  77:         asset = asset_;

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L77

```solidity
src\vault\adapter\beefy\BeefyAdapter.sol:
  46     function initialize(
  73:         beefyVault = IBeefyVault(_beefyVault);
  74:         beefyBooster = IBeefyBooster(_beefyBooster);

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L73-L74

```solidity
src\vault\adapter\yearn\YearnAdapter.sol:
  34     function initialize(
  45:         yVault = VaultAPI(IYearnRegistry(externalRegistry).latestVault(_asset));

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L45


### [G-04] Using ``delete``  instead of setting mapping/state variable ``0`` saves gas

2 results - 2 files:
```solidity
src\utils\MultiRewardStaking.sol:

  186:       accruedRewards[user][_rewardTokens[i]] = 0;

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L186


```solidity
src\vault\adapter\abstracts\AdapterBase.sol:

  577:         underlyingBalance = 0;

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L577


**Recommendation code:**
```diff
src\vault\adapter\abstracts\AdapterBase.sol:

- 577:         underlyingBalance = 0;
+ 577:         delete underlyingBalance;

```

### [G-05] Using `storage` instead of `memory` for `structs/arrays` saves gas

When fetching data from a storage location, assigning the data to a ``memory`` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the ``memory`` keyword, declaring the variable with the ``storage`` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a ``memory`` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires ``memory``, or if the array/struct is being read from another ``memory`` array/struct

7 results - 2 files:
```solidity

src\utils\MultiRewardEscrow.sol:

  157:    Escrow memory escrow = escrows[escrowId];
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L157


```solidity
src\utils\MultiRewardStaking.sol:

  176:    EscrowInfo memory escrowInfo = escrowInfos[_rewardTokens[i]];

  254:    RewardInfo memory rewards = rewardInfos[rewardToken];

  297:    RewardInfo memory rewards = rewardInfos[rewardToken];

  328:    RewardInfo memory rewards = rewardInfos[rewardToken];

  375:    RewardInfo memory rewards = rewardInfos[rewardToken];

  414:    RewardInfo memory rewards = rewardInfos[_rewardToken];
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L176


### [G-06] Add ``unchecked {}`` for subtractions where the operands cannot underflow because of a previous ``require`` or ``if`` statement

```
require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a } 
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
```
This will stop the check for overflow and underflow so it will save gas.

3 results - 1 file:
```solidity
src\utils\MultiRewardStaking.sol:
 
  356:     if (rewardsEndTimestamp > block.timestamp)
  357       amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);

  392:     if (rewards.rewardsEndTimestamp > block.timestamp) {
  393       elapsed = block.timestamp - rewards.lastUpdatedTimestamp;

  394:     } else if (rewards.rewardsEndTimestamp > rewards.lastUpdatedTimestamp) {
  395       elapsed = rewards.rewardsEndTimestamp - rewards.lastUpdatedTimestamp;
  396     }

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L356-L357
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L392-L396


```solidity
src\vault\adapter\abstracts\AdapterBase.sol:

  535:             performanceFee_ > 0 && shareValue > highWaterMark_
  536                 ? performanceFee_.mulDiv(
  537                     (shareValue - highWaterMark_) * totalSupply(),
  538                     1e36,
  539                     Math.Rounding.Down
  540                 )
  541                 : 0;

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L534-L541

```solidity
src\vault\adapter\yearn\YearnAdapter.sol:

  150:         if (assets >= _depositLimit) return 0;
  151         return _depositLimit - assets;

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L150-L151


### [G-07] Use ``assembly`` to write _address storage values_ 

19 results - 5 files:
```solidity
src\utils\MultiRewardEscrow.sol:

  30      constructor(address _owner, address _feeRecipient) Owned(_owner) {
  31:        feeRecipient = _feeRecipient;

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L31

```solidity
src\vault\DeploymentController.sol:

  35      constructor(
  41:        cloneFactory = _cloneFactory;
  42:        cloneRegistry = _cloneRegistry;
  43:        templateRegistry = _templateRegistry;

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L41-L43

```solidity
src\vault\VaultController.sol:

  53      constructor(
  61:        adminProxy = _adminProxy;
  62:        vaultRegistry = _vaultRegistry;
  63:        permissionRegistry = _permissionRegistry;
  64:        escrow = _escrow;

  836     function _setDeploymentController(IDeploymentController _deploymentController) internal {
  842:       deploymentController = _deploymentController;

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L61-L64

```solidity
src\utils\MultiRewardStaking.sol:

  41      function initialize(
  53:        escrow = _escrow;

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L53

```solidity
src\vault\Vault.sol:

  57      function initialize(
  77:         asset = asset_;
  78:         adapter = adapter_;
  88:         fees = fees_;
  91:         feeRecipient = feeRecipient_;

  525     function proposeFees(VaultFees calldata newFees) external onlyOwner {
  533:         proposedFees = newFees;

  540     function changeFees() external {
  545:         fees = proposedFees;

  553     function setFeeRecipient(address _feeRecipient) external onlyOwner {
  558:         feeRecipient = _feeRecipient;

  578     function proposeAdapter(IERC4626 newAdapter) external onlyOwner {
  582:         proposedAdapter = newAdapter;

  594     function changeAdapter() external takeFees {
  608:         adapter = proposedAdapter;

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L77-L78

**Recommendation Code:**
```diff

src\vault\Vault.sol:

  57      function initialize(
- 77:         asset = asset_;
+                  assembly {                      
+                      sstore(asset.slot, asset _)
+                  }                               
          
```


### [G-08] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html 

Use a larger size then downcast where needed.

4 results - 4 files:
```solidity

src\utils\MultiRewardStaking.sol:

//@audit uint8 _decimals
  51:     _decimals = IERC20Metadata(address(_stakingToken)).decimals();
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L51

```solidity
src\vault\Vault.sol:

//@audit uint8 _decimals
  82:         _decimals = IERC20Metadata(address(asset_)).decimals();
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L82

```solidity
src\vault\adapter\abstracts\AdapterBase.sol:

//@audit uint8 _decimals
  77:         _decimals = IERC20Metadata(asset).decimals();
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L77


```solidity
src\utils\MultiRewardStaking.sol:

//@audit uint160 rewardsPerSecond
  359:     return (block.timestamp + (amount / uint256(rewardsPerSecond))).safeCastTo32();
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L359


### [G-09] Don't use _msgSender() if not supporting EIP-2771

Use ``msg.sender`` if the code does not implement EIP-2771 trusted forwarder support.

Reference: https://eips.ethereum.org/EIPS/eip-2771

4 results - 1 file:
```solidity
src\vault\adapter\abstracts\AdapterBase.sol:

  119:    _deposit(_msgSender(), receiver, assets, shares);

  138:    _deposit(_msgSender(), receiver, assets, shares);
 
  182:    _withdraw(_msgSender(), receiver, owner, assets, shares);
 
  201:    _withdraw(_msgSender(), receiver, owner, assets, shares);
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L119


### [G-10] ``internal`` functions only called once can be inlined to save gas

Not inlining costs **20** to **40** gas because of two extra ``JUMP`` instructions and additional stack operations needed for function calls.

4 results - 2 files:

```solidity
src\utils\MultiRewardStaking.sol:

  191   function _lockToken(
  192     address user,
  193     IERC20 rewardToken,
  194     uint256 rewardAmount,
  195     EscrowInfo memory escrowInfo
  196:   ) internal {

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L191-L202

```solidity
src\vault\VaultController.sol:

  141   function _registerCreatedVault(
  142     address vault,
  143     address staking,
  144     VaultMetadata memory metadata
  145:   ) internal {

  154:   function _handleVaultStakingRewards(address vault, bytes memory rewardsData) internal {

  692:   function _verifyAdapterConfiguration(address adapter, bytes32 adapterId) internal view {
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L141-L151


### [G-11] Emitting storage values instead of the memory one

Reading the values published below from ``memory`` instead of ``storage`` saves gas.

1 result - 1 file:
```solidity
src\vault\Vault.sol:
  629:     function setQuitPeriod(uint256 _quitPeriod) external onlyOwner {
  630:         if (_quitPeriod < 1 days || _quitPeriod > 7 days)
  631:             revert InvalidQuitPeriod();
  632: 
  633:         quitPeriod = _quitPeriod;
  634: 
  635:         emit QuitPeriodSet(quitPeriod);
  636:     }

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L629-L636

```diff
src\vault\Vault.sol:
  629:     function setQuitPeriod(uint256 _quitPeriod) external onlyOwner {
  630:         if (_quitPeriod < 1 days || _quitPeriod > 7 days)
  631:             revert InvalidQuitPeriod();
  632: 
  633:         quitPeriod = _quitPeriod;
  634: 
- 635:         emit QuitPeriodSet(quitPeriod);
+ 635:         emit QuitPeriodSet(_quitPeriod);
  636:     }

```

### [G-12] Superfluous event fields

``block.timestamp`` and ``block.number`` are added to event information by default so adding them manually wastes gas.

```solidity
2 result - 1 file:

src\vault\Vault.sol:

  512:    event NewFeesProposed(VaultFees newFees, uint256 timestamp);

  569:    event NewAdapterProposed(IERC4626 newAdapter, uint256 timestamp);

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L512
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L569



### [G-13] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}). Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.

7 results - 4 files:
```solidity
src\utils\EIP165.sol:

  7:     function supportsInterface(bytes4 interfaceId) public view virtual returns (bool) {}
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/EIP165.sol#L7

```solidity
src\vault\Vault.sol:

  473    function takeManagementAndPerformanceFees()
  474        external
  475        nonReentrant
  476        takeFees
  477:    {}
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L477

```solidity
src\vault\adapter\abstracts\AdapterBase.sol:

  258:   function _totalAssets() internal view virtual returns (uint256) {}

  265:   function _underlyingBalance() internal view virtual returns (uint256) {}

  271    function convertToUnderlyingShares(uint256 assets, uint256 shares)
  272        public
  273        view
  274        virtual
  275        returns (uint256)
  276:    {}
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L258

```solidity
src\vault\adapter\abstracts\WithRewards.sol:

  13:    function rewardTokens() external view virtual returns (address[] memory) {}

  15:    function claim() public virtual onlyStrategy {}
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/WithRewards.sol#L13


### [G-14] Setting the _constructor_ to `payable` (117 gas)

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of ```msg.value == 0``` and saves ```13 gas``` on deployment with no security risks.

9 results - 9 files:
```solidty
src\utils\MultiRewardEscrow.sol:

  30:   constructor(address _owner, address _feeRecipient) Owned(_owner) {
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L30


```solidty
src\vault\AdminProxy.sol:

  16:   constructor(address _owner) Owned(_owner) {}
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L16


```solidty
src\vault\CloneFactory.sol:

  23:   constructor(address _owner) Owned(_owner) {}
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneFactory.sol#L23


```solidty
src\vault\CloneRegistry.sol:

  22:   constructor(address _owner) Owned(_owner) {}
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L22

 
```solidty
src\vault\DeploymentController.sol:

  35:   constructor(
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L35


```solidty
src\vault\PermissionRegistry.sol:

  20:   constructor(address _owner) Owned(_owner) {}
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L20


```solidty
src\vault\TemplateRegistry.sol:

  24:   constructor(address _owner) Owned(_owner) {}
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L24  


```solidty
src\vault\VaultController.sol:

  53:   constructor(
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L53


**Recommendation:**
Set the constructor to ```payable```


### [G-15] `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables

2 results - 2 files:
```solidity

src\vault\adapter\abstracts\AdapterBase.sol:

  158:    underlyingBalance += _underlyingBalance() - underlyingBalance_;

  225:    underlyingBalance -= underlyingBalance_ - _underlyingBalance();
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L158
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L225


`x += y (x -= y)` costs more gas than `x = x  + y (x = x - y)` for state variables.


### [G-16] ``>=`` costs less gas than ``>``

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas

4 results - 3 files:
```solidity
src\utils\MultiRewardStaking.sol:

  309:       prevEndTime > block.timestamp ? prevEndTime : block.timestamp.safeCastTo32(),
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L309

```solidity
src\vault\Vault.sol:

  431         return
  432:             managementFee > 0
  433                 ? managementFee.mulDiv(
  434                     totalAssets() * (block.timestamp - feesUpdatedAt),
  435                     SECONDS_PER_YEAR,
  436                     Math.Rounding.Down
  437                 ) / 1e18
  438                 : 0;

  452         return
  453:             performanceFee > 0 && shareValue > highWaterMark
  454                 ? performanceFee.mulDiv(
  455                     (shareValue - highWaterMark) * totalSupply(),
  456                     1e36,
  457                     Math.Rounding.Down
  458                 )
  459                 : 0;
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L431-L438
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L452-L459

```solidity
src\vault\adapter\abstracts\AdapterBase.sol:

  534         return
  535:             performanceFee_ > 0 && shareValue > highWaterMark_
  536                 ? performanceFee_.mulDiv(
  537                     (shareValue - highWaterMark_) * totalSupply(),
  538                     1e36,
  539                     Math.Rounding.Down
  540                 )
  541                 : 0;
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L534-L541


### [G-17] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

2 results - 2 files:
```solidity
src\vault\PermissionRegistry.sol:

  43:    if (newPermissions[i].endorsed && newPermissions[i].rejected) revert Mismatch();
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L43

```solidity
src\vault\Vault.sol:

  490:   if (totalFee > 0 && currentAssets > 0)
  491             _mint(feeRecipient, convertToShares(totalFee));
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L490


**Recomendation Code:**
```diff
src\vault\Vault.sol:

- 490:   if (totalFee > 0 && currentAssets > 0)
+           if (totalFee > 0)
+           if (currentAssets > 0) {
  491             _mint(feeRecipient, convertToShares(totalFee));
+              }
+          }

```

### [G-18] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses ```OR/AND``` logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 

//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```
5 results - 4 files:
```solidity
src\utils\MultiRewardStaking.sol:

  481:    if (recoveredAddress == address(0) || recoveredAddress != owner) revert InvalidSigner(recoveredAddress);
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L481

```solidity
src\vault\Vault.sol:

  702:    if (recoveredAddress == address(0) || recoveredAddress != owner)
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L702

```solidity
src\vault\VaultController.sol:

  669:    if (msg.sender != metadata.creator || msg.sender != owner) revert NotSubmitterNorOwner(msg.sender);
  
  837:    if (address(_deploymentController) == address(0) || address(deploymentController) == address(_deploymentController))
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L669

```solidity
src\vault\adapter\abstracts\AdapterBase.sol:

  670:    if (recoveredAddress == address(0) || recoveredAddress != owner)
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L670


### [G-19] Optimize names to save gas (22 gas)

Contracts most called functions could simply save gas by function ordering via ```Method ID```. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because ```22 gas``` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

**Context:** 
All Contracts


**Recommendation:** 
Find a lower ```method ID``` name for the most called functions for example Call() vs. Call1() is cheaper by ```22 gas```
For example, the function IDs in the ```VaultController.sol``` contract will be the most used; A lower method ID may be given.

**Proof of Consept:**
https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

VaultController.sol function names can be named and sorted according to METHOD ID

```js
Sighash   |   Function Signature
========================
68432477  =>  setPermissions(address[],Permission[])
69e3b3dc  =>  deployVault(VaultInitParams,DeploymentArgs,DeploymentArgs,address,bytes,VaultMetadata,uint256)
9cfc553d  =>  _deployVault(VaultInitParams,IDeploymentController)
6cfc1202  =>  _registerCreatedVault(address,address,VaultMetadata)
459f12e4  =>  _handleVaultStakingRewards(address,bytes)
71b7ae93  =>  _handleInitialDeposit(uint256,IERC20,IERC4626)
fc263a50  =>  deployAdapter(IERC20,DeploymentArgs,DeploymentArgs,uint256)
96c2b2bd  =>  _deployAdapter(IERC20,DeploymentArgs,DeploymentArgs,IDeploymentController)
f1959d73  =>  __deployAdapter(DeploymentArgs,bytes,IDeploymentController)
8515c57f  =>  _encodeAdapterData(DeploymentArgs,bytes)
5b4c2524  =>  _deployStrategy(DeploymentArgs,IDeploymentController)
e4f49a99  =>  deployStaking(IERC20)
c461a414  =>  _deployStaking(IERC20,IDeploymentController)
40a2c775  =>  proposeVaultAdapters(address[],IERC4626[])
5c456e16  =>  changeVaultAdapters(address[])
0027fdc7  =>  proposeVaultFees(address[],VaultFees[])
3c27cc18  =>  changeVaultFees(address[])
6cd77c1b  =>  _registerVault(address,VaultMetadata)
ccc0f886  =>  addStakingRewardsTokens(address[],bytes[])
87b31809  =>  changeStakingRewardsSpeeds(address[],IERC20[],uint160[])
26699bcc  =>  fundStakingRewards(address[],IERC20[],uint256[])
ecd176d1  =>  setEscrowTokenFees(IERC20[],uint256[])
5dcc700f  =>  addTemplateCategories(bytes32[])
7bd0703d  =>  toggleTemplateEndorsements(bytes32[],bytes32[])
a52f0ead  =>  pauseAdapters(address[])
8ba684e0  =>  pauseVaults(address[])
6d956763  =>  unpauseAdapters(address[])
2c3f064f  =>  unpauseVaults(address[])
0bbc1ffc  =>  _verifyCreatorOrOwner(address)
c7db9835  =>  _verifyCreator(address)
5cbea48b  =>  _verifyToken(address)
33785a9f  =>  _verifyAdapterConfiguration(address,bytes32)
23f4c026  =>  _verifyEqualArrayLength(uint256,uint256)
0a95d1ff  =>  nominateNewAdminProxyOwner(address)
e45e1b94  =>  acceptAdminProxyOwnership()
70897b23  =>  setPerformanceFee(uint256)
d7fe6a85  =>  setAdapterPerformanceFees(address[])
d643ad32  =>  setHarvestCooldown(uint256)
23e45a62  =>  setAdapterHarvestCooldowns(address[])
eab63b2a  =>  setDeploymentController(IDeploymentController)
46ef81f4  =>  _setDeploymentController(IDeploymentController)
ebf0f0db  =>  setActiveTemplateId(bytes32,bytes32)
```

### [G-20] External visibility can be used in public functions

The appearance of the following functions can be changed from public to external.

13 results - 5 files:
```solidity
src\utils\EIP165.sol:

  7:   function supportsInterface(bytes4 interfaceId) public view virtual returns (bool) {}
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/EIP165.sol#L7

```solidity
src\vault\Vault.sol:

  124:    function deposit(uint256 assets) public returns (uint256) {

  200:    function withdraw(uint256 assets) public returns (uint256) {

  211     function withdraw(
  212         uint256 assets,
  213         address receiver,
  214         address owner
  215:     ) public nonReentrant syncFeeCheckpoint returns (uint256 shares) {

  253     function redeem(
  254         uint256 shares,
  255         address receiver,
  256         address owner
  257:     ) public nonReentrant returns (uint256 assets) {

  340:    function previewMint(uint256 shares) public view returns (uint256 assets) {

  664     function permit(
  665         address owner,
  666         address spender,
  667         uint256 value,
  668         uint256 deadline,
  669         uint8 v,
  670         bytes32 r,
  671         bytes32 s
  672:     ) public virtual {
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L124

```solidity
src\vault\adapter\abstracts\AdapterBase.sol:

  173     function withdraw(
  174         uint256 assets,
  175         address receiver,
  176         address owner
  177:     ) public virtual override returns (uint256) {

  193     function redeem(
  194         uint256 shares,
  195         address receiver,
  196         address owner
  197:     ) public virtual override returns (uint256) {

  549:    function setPerformanceFee(uint256 newFee) public onlyOwner {
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L173

```solidity
src\vault\adapter\abstracts\WithRewards.sol:

  15:     function claim() public virtual onlyStrategy {}

  21:     function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/WithRewards.sol#L15

```solidity
src\vault\adapter\yearn\YearnAdapter.sol:

  144:    function maxDeposit(address) public view override returns (uint256) {
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L144

### [G-21] Use a more recent version of solidity

In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

All contracts in scope (**35 files**) are written in ``pragma solidity ^0.8.15;`` and I recommend using the newer battle-tested version of Solidity ``0.8.17``.


**Context:**
All contracts

### [G-22] Cheaper to calculate domain separator every time

Since INITIAL_CHAIN_ID and INITIAL_DOMAIN_SEPARATOR are no longer immutable, but are state variables, you end up looking up their value every time, which incurs a very large gas penalty. 
 
Use OpenZeppelin's version which is optimized for this case.
https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC20/extensions/draft-ERC20PermitUpgradeable.sol

3 results - 3 files:

```solidity
src\utils\MultiRewardStaking.sol:

  488:     return block.chainid == INITIAL_CHAIN_ID ? INITIAL_DOMAIN_SEPARATOR : computeDomainSeparator();

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L488

```solidity
src\vault\Vault.sol:

  710:         return
  711:             block.chainid == INITIAL_CHAIN_ID
  712:                 ? INITIAL_DOMAIN_SEPARATOR
  713:                 : computeDomainSeparator();
  714:     }
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L710-L714

```solidity
src\vault\adapter\abstracts\AdapterBase.sol:

  678:         return
  679:             block.chainid == INITIAL_CHAIN_ID
  680:                 ? INITIAL_DOMAIN_SEPARATOR
  681:                 : computeDomainSeparator();
  682:     }

```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L678-L682

### [G-23] Use hardcode address instead ``address(this)``

Instead of ``address(this)``, it is more gas-efficient to pre-calculate and use the address with a hardcode. Foundry's ``script.sol`` and solmate````LibRlp.sol`` contracts can do this.

Reference:
https://book.getfoundry.sh/reference/forge-std/compute-create-address
https://twitter.com/transmissions11/status/1518507047943245824


20 results - 8 files
```solidity
src\utils\MultiRewardEscrow.sol:

  100:     token.safeTransferFrom(msg.sender, address(this), amount);
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L100


```solidity

src\utils\MultiRewardStaking.sol:

  113:     IERC20(asset()).safeTransferFrom(caller, address(this), assets);

  259:       rewardToken.safeTransferFrom(msg.sender, address(this), amount);

  305:     uint256 remainder = rewardToken.balanceOf(address(this));

  334:     rewardToken.safeTransferFrom(msg.sender, address(this), amount);
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L113
 
```solidity
src\vault\Vault.sol:

  153:         asset.safeTransferFrom(msg.sender, address(this), assets);
  154  
  155:         adapter.deposit(assets, address(this));

  193:         asset.safeTransferFrom(msg.sender, address(this), assets);
  194  
  195:         adapter.deposit(assets, address(this));

  237:         adapter.withdraw(assets, receiver, address(this));

  275:         adapter.withdraw(assets, receiver, address(this));

  286:         return adapter.convertToAssets(adapter.balanceOf(address(this)));

  612:         adapter.deposit(asset.balanceOf(address(this)), address(this));
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L153


```solidity
src\vault\VaultController.sol:

  170:       asset.safeTransferFrom(msg.sender, address(this), initialDeposit);
 
  526:       rewardTokens[i].transferFrom(msg.sender, address(this), amounts[i]);
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L170

```solidity
src\vault\adapter\abstracts\AdapterBase.sol:

  153:         IERC20(asset()).safeTransferFrom(caller, address(this), assets);
```


```solidity
src\vault\adapter\abstracts\OnlyStrategy.sol:

  11:     if (msg.sender != address(this)) revert NotStrategy(msg.sender);
```


```solidity
src\vault\adapter\beefy\BeefyAdapter.sol:
 
  118:         return beefyBalanceCheck.balanceOf(address(this));

  199:             beefyBooster.stake(beefyVault.balanceOf(address(this)));
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L118

```solidity
src\vault\adapter\yearn\YearnAdapter.sol:

  85:         return yVault.balanceOf(address(this));
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L85
