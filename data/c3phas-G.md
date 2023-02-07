### Table of Contents
- [FINDINGS](#findings)
- [Using immutable on variables that are only set in the constructor and never after (Save 16.8K gas)](#using-immutable-on-variables-that-are-only-set-in-the-constructor-and-never-after-save-168k-gas)
- [Cheaper to calculate domain separator every time(12.6k gas)](#cheaper-to-calculate-domain-separator-every-time126k-gas)
  - [Vault.sol.DOMAIN\_SEPARATOR(): Save 4.2K gas](#vaultsoldomain_separator-save-42k-gas)
  - [MultiRewardStaking.sol.DOMAIN\_SEPARATOR(): Save 4.2K gas](#multirewardstakingsoldomain_separator-save-42k-gas)
  - [AdapterBase.sol.DOMAIN\_SEPARATOR(): Save 4.2K gas](#adapterbasesoldomain_separator-save-42k-gas)
- [Tighly pack storage variables/optimize the order of variable declaration(Save 6.3K gas)](#tighly-pack-storage-variablesoptimize-the-order-of-variable-declarationsave-63k-gas)
  - [Pack `feesUpdatedAt` with `asset` to save 1 SLOT (2.1k gas)](#pack-feesupdatedat-with-asset-to-save-1-slot-21k-gas)
  - [Pack `proposedFeeTime` with `feeRecipient` to save 1 SLOT(2.1k gas)](#pack-proposedfeetime-with-feerecipient-to-save-1-slot21k-gas)
  - [Pack `proposedAdapterTime` with `proposedAdapter` to save 1 SLOT(2.1k gas)](#pack-proposedadaptertime-with-proposedadapter-to-save-1-slot21k-gas)
- [The result of a function call should be cached rather than re-calling the function](#the-result-of-a-function-call-should-be-cached-rather-than-re-calling-the-function)
  - [YearnAdapter.sol.\_shareValue():  Results of yVault.totalSupply() should be cached(Saves gas in happy case)](#yearnadaptersol_sharevalue--results-of-yvaulttotalsupply-should-be-cachedsaves-gas-in-happy-case)
  - [YearnAdapter.sol.initialize(): Results of asset() should be cached and the cached value used instead of calling it twice](#yearnadaptersolinitialize-results-of-asset-should-be-cached-and-the-cached-value-used-instead-of-calling-it-twice)
  - [BeefyAdapter.sol.initialize(): Results of asset() should be cached and the cached value used](#beefyadaptersolinitialize-results-of-asset-should-be-cached-and-the-cached-value-used)
- [Use the cached value instead of fetching a storage value](#use-the-cached-value-instead-of-fetching-a-storage-value)
  - [Vault.sol.accruedPerformanceFee(): highWaterMark has already been cached and thus the cached value should be used. Saves ~200 gas(2 SLOADS)](#vaultsolaccruedperformancefee-highwatermark-has-already-been-cached-and-thus-the-cached-value-should-be-used-saves-200-gas2-sloads)
  - [Vault.sol.initialize(): asset has been cached already and should be used here(Save 1 SLOAD: ~100 gas)](#vaultsolinitialize-asset-has-been-cached-already-and-should-be-used-heresave-1-sload-100-gas)
- [Internal/Private functions only called once can be inlined to save gas](#internalprivate-functions-only-called-once-can-be-inlined-to-save-gas)
- [Multiple accesses of a mapping/array should use a local variable cache](#multiple-accesses-of-a-mappingarray-should-use-a-local-variable-cache)
  - [TemplateRegistry.sol.addTemplateCategory(): templateCategoryExists\[templateCategory\] should be cached in local storage](#templateregistrysoladdtemplatecategory-templatecategoryexiststemplatecategory-should-be-cached-in-local-storage)
  - [MultiRewardStaking.sol.\_accrueRewards(): rewardInfos\[\_rewardToken\] should be cached in local storage](#multirewardstakingsol_accruerewards-rewardinfos_rewardtoken-should-be-cached-in-local-storage)
- [Emitting storage values instead of the memory one.(Save ~200 gas)](#emitting-storage-values-instead-of-the-memory-onesave-200-gas)
  - [Vault.sol.setQuitPeriod(): emit `_quitPeriod` : saves 1 SLOAD: ~100 gas](#vaultsolsetquitperiod-emit-_quitperiod--saves-1-sload-100-gas)
  - [Vault.sol.initialize(): asset has been cached already and cached value should be emitted here(Save 1 SLOAD : ~100 gas)](#vaultsolinitialize-asset-has-been-cached-already-and-cached-value-should-be-emitted-heresave-1-sload--100-gas)
- [Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate](#multiple-address-mappings-can-be-combined-into-a-single-mapping-of-an-address-to-a-struct-where-appropriate)
- [Using storage instead of memory for structs/arrays saves gas](#using-storage-instead-of-memory-for-structsarrays-saves-gas)
- [IF's/require() statements that check input arguments should be at the top of the function](#ifsrequire-statements-that-check-input-arguments-should-be-at-the-top-of-the-function)
  - [cheaper to reorder the if's check to have the cheap check for functional parameter before other operations](#cheaper-to-reorder-the-ifs-check-to-have-the-cheap-check-for-functional-parameter-before-other-operations)
  - [Move the functional parameter check to be the first operation](#move-the-functional-parameter-check-to-be-the-first-operation)
  - [Move the if for functional parameter to the top](#move-the-if-for-functional-parameter-to-the-top)
- [`keccak256()` should only need to be called on a specific string literal once](#keccak256-should-only-need-to-be-called-on-a-specific-string-literal-once)
- [x += y costs more gas than x = x + y for state variables](#x--y-costs-more-gas-than-x--x--y-for-state-variables)
- [Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead](#usage-of-uintsints-smaller-than-32-bytes-256-bits-incurs-overhead)
- [Using unchecked blocks to save gas](#using-unchecked-blocks-to-save-gas)
- [Using unchecked blocks to save gas - Increments in for loop can be unchecked  ( save 30-40 gas per loop iteration)](#using-unchecked-blocks-to-save-gas---increments-in-for-loop-can-be-unchecked---save-30-40-gas-per-loop-iteration)

## FINDINGS
NB: Some functions have been truncated where necessary to just show affected parts of the code
Through out the report some places might be denoted with audit tags to show the actual place affected.

## Using immutable on variables that are only set in the constructor and never after (Save 16.8K gas)
Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD.
Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas)

`Gas Per variable: 2.1k`

`Total Instances: 8 `

**`Gas Saved: 8 * 2.1k=16.8k`**


https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L191
```solidity
File: /src/utils/MultiRewardEscrow.sol
191:  address public feeRecipient;
```

```diff
diff --git a/src/utils/MultiRewardEscrow.sol b/src/utils/MultiRewardEscrow.sol
index cf50b08..67744e0 100644
--- a/src/utils/MultiRewardEscrow.sol
+++ b/src/utils/MultiRewardEscrow.sol
@@ -188,7 +188,7 @@ contract MultiRewardEscrow is Owned {
                             FEE LOGIC
     //////////////////////////////////////////////////////////////*/

-  address public feeRecipient;
+  address public immutable feeRecipient;
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/DeploymentController.sol#L23-L25

```solidity
File: /src/vault/DeploymentController.sol
23:  ICloneFactory public cloneFactory;
24:  ICloneRegistry public cloneRegistry;
25:  ITemplateRegistry public templateRegistry;
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L535
```solidity
File: /src/vault/VaultController.sol
535:  IMultiRewardEscrow public escrow;
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L717
```solidity
File: /src/vault/VaultController.sol
717:  IAdminProxy public adminProxy;
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L387
```solidity
File: /src/vault/VaultController.sol
387:  IVaultRegistry public vaultRegistry;
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L822
```solidity
File: /src/vault/VaultController.sol
822:  IPermissionRegistry public permissionRegistry;
```
## Cheaper to calculate domain separator every time(12.6k gas)

Since `INITIAL_CHAIN_ID` and `INITIAL_DOMAIN_SEPARATOR` are no longer immutable, but are state variables, you end up looking up their value every time, which incurs a very large gas penalty. It's cheaper to just calculate it every time.

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L709-L714
### Vault.sol.DOMAIN_SEPARATOR(): Save 4.2K gas 
```solidity
File: /src/vault/Vault.sol
709:    function DOMAIN_SEPARATOR() public view returns (bytes32) {
710:        return
711:            block.chainid == INITIAL_CHAIN_ID
712:                ? INITIAL_DOMAIN_SEPARATOR
713:                : computeDomainSeparator();
714:    }
```


https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L487-L489
### MultiRewardStaking.sol.DOMAIN_SEPARATOR(): Save 4.2K gas 
```solidity
File: /src/utils/MultiRewardStaking.sol
487:  function DOMAIN_SEPARATOR() public view returns (bytes32) {
488:    return block.chainid == INITIAL_CHAIN_ID ? INITIAL_DOMAIN_SEPARATOR : computeDomainSeparator();
489:  }
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L677-L682
### AdapterBase.sol.DOMAIN_SEPARATOR(): Save 4.2K gas 
```solidity
File: /src/vault/adapter/abstracts/AdapterBase.sol
677:    function DOMAIN_SEPARATOR() public view virtual returns (bytes32) {
678:        return
679:            block.chainid == INITIAL_CHAIN_ID
680:                ? INITIAL_DOMAIN_SEPARATOR
681:                : computeDomainSeparator();
682:    }
```

## Tighly pack storage variables/optimize the order of variable declaration(Save 6.3K gas)
**Note the following lines and the explanation to understand the why and how the packing would be achieved**

**This packing is only achievable as we can reduce the size of time variable from uint256 to uint64 which should be safe as they are just timestamps**

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L468
```solidity
File: /src/vault/Vault.sol
468:    uint256 public feesUpdatedAt;

508:    uint256 public proposedFeeTime;

567:    uint256 public proposedAdapterTime;
```
 As `feesUpdatedAt,proposedFeeTime,proposedAdapterTime` are  simply  variables representing timestamps, we could get away with making them to be of type  `uint64` which should be safe for around `532 years`
 
### Pack `feesUpdatedAt` with `asset` to save 1 SLOT (2.1k gas)
For `feesUpdatedAt`  we could pack it with `    IERC20 public asset;` on [Line 37](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L37) 
```diff
diff --git a/src/vault/Vault.sol b/src/vault/Vault.sol
index 7a8f941..4bce208 100644
--- a/src/vault/Vault.sol
+++ b/src/vault/Vault.sol
@@ -35,6 +35,7 @@ contract Vault is
     uint256 constant SECONDS_PER_YEAR = 365.25 days;

     IERC20 public asset;
+    uint64 public feesUpdatedAt;
     uint8 internal _decimals;

     bytes32 public contractName;
@@ -465,7 +466,6 @@ contract Vault is

     uint256 public highWaterMark = 1e18;
     uint256 public assetsCheckpoint;
-    uint256 public feesUpdatedAt;

```

### Pack `proposedFeeTime` with `feeRecipient` to save 1 SLOT(2.1k gas)
Change `proposedFeeTime` to a `uint64` and pack it with address `feeRecipient` on [Line 510](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L510)

```diff
diff --git a/src/vault/Vault.sol b/src/vault/Vault.sol
index 7a8f941..a71094b 100644
--- a/src/vault/Vault.sol
+++ b/src/vault/Vault.sol
@@ -505,9 +505,9 @@ contract Vault is
     VaultFees public fees;

     VaultFees public proposedFees;
-    uint256 public proposedFeeTime;

     address public feeRecipient;
+    uint64 public proposedFeeTime;
```

### Pack `proposedAdapterTime` with `proposedAdapter` to save 1 SLOT(2.1k gas)
Change `proposedAdapterTime` to a `uint64` and pack it with IERC4626 `proposedAdapter` on [Line 566](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L566)

```diff
diff --git a/src/vault/Vault.sol b/src/vault/Vault.sol
index 7a8f941..da2c9a4 100644
--- a/src/vault/Vault.sol
+++ b/src/vault/Vault.sol
@@ -564,7 +564,7 @@ contract Vault is

     IERC4626 public adapter;
     IERC4626 public proposedAdapter;
-    uint256 public proposedAdapterTime;
+    uint64 public proposedAdapterTime;

```

## The result of a function call should be cached rather than re-calling the function

External calls are expensive. Consider caching the following:

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L89-L98
### YearnAdapter.sol.\_shareValue():  Results of yVault.totalSupply() should be cached(Saves gas in happy case)
```solidity
File: /src/vault/adapter/yearn/YearnAdapter.sol
89:    function _shareValue(uint256 yShares) internal view returns (uint256) {
90:        if (yVault.totalSupply() == 0) return yShares; //@audit: initial call

92:        return
93:            yShares.mulDiv(
94:                _freeFunds(),
95:                yVault.totalSupply(),//@audit: 2nd call
96:                Math.Rounding.Down
97:            );
98:    }
```

```diff
diff --git a/src/vault/adapter/yearn/YearnAdapter.sol b/src/vault/adapter/yearn/YearnAdapter.sol
index d951e63..12114a3 100644
--- a/src/vault/adapter/yearn/YearnAdapter.sol
+++ b/src/vault/adapter/yearn/YearnAdapter.sol
@@ -87,12 +87,13 @@ contract YearnAdapter is AdapterBase {

     /// @notice Determines the current value of `yShares` in assets
     function _shareValue(uint256 yShares) internal view returns (uint256) {
-        if (yVault.totalSupply() == 0) return yShares;
+        uint256 _yVaultTotalSupply = yVault.totalSupply();
+        if (_yVaultTotalSupply == 0) return yShares;

         return
             yShares.mulDiv(
                 _freeFunds(),
-                yVault.totalSupply(),
+                _yVaultTotalSupply,
                 Math.Rounding.Down
             );
     }
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L34-L55
### YearnAdapter.sol.initialize(): Results of asset() should be cached and the cached value used instead of calling it twice
```solidity
File: /src/vault/adapter/yearn/YearnAdapter.sol
34:    function initialize(

47:        _name = string.concat(
48:            "Popcorn Yearn",
49:            IERC20Metadata(asset()).name(),//@audit: 1st call
50:            " Adapter"
51:        );
52:        _symbol = string.concat("popY-", IERC20Metadata(asset()).symbol());//@audit: 2nd call
```

```diff
diff --git a/src/vault/adapter/yearn/YearnAdapter.sol b/src/vault/adapter/yearn/YearnAdapter.sol
index d951e63..f69ad26 100644
--- a/src/vault/adapter/yearn/YearnAdapter.sol
+++ b/src/vault/adapter/yearn/YearnAdapter.sol
@@ -44,12 +44,14 @@ contract YearnAdapter is AdapterBase {

         yVault = VaultAPI(IYearnRegistry(externalRegistry).latestVault(_asset));

+        address asset_ = asset();
+
         _name = string.concat(
             "Popcorn Yearn",
-            IERC20Metadata(asset()).name(),
+            IERC20Metadata(asset_).name(),
             " Adapter"
         );
-        _symbol = string.concat("popY-", IERC20Metadata(asset()).symbol());
+        _symbol = string.concat("popY-", IERC20Metadata(asset_).symbol());

         IERC20(_asset).approve(address(yVault), type(uint256).max);
     }
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L46-L84
### BeefyAdapter.sol.initialize(): Results of asset() should be cached and the cached value used
```solidity
File: /src/vault/adapter/beefy/BeefyAdapter.sol
46:    function initialize(
47:        bytes memory adapterInitData,
48:        address registry,
49:        bytes memory beefyInitData
50:    ) external initializer {

59:        if (IBeefyVault(_beefyVault).want() != asset()) //@audit: 1st call
60:            revert InvalidBeefyVault(_beefyVault);

66:        _name = string.concat(
67:            "Popcorn Beefy",
68:            IERC20Metadata(asset()).name(), //@audit: 2nd call
69:            " Adapter"
70:        );
71:        _symbol = string.concat("popB-", IERC20Metadata(asset()).symbol());//@audit: 3rd call

80:        IERC20(asset()).approve(_beefyVault, type(uint256).max);//@audit: 4th call
```


## Use the cached value instead of fetching a storage value 
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L447-L460
### Vault.sol.accruedPerformanceFee(): highWaterMark has already been cached and thus the cached value should be used. Saves ~200 gas(2 SLOADS)
```solidity
File: /src/vault/Vault.sol
447:    function accruedPerformanceFee() public view returns (uint256) {
448:        uint256 highWaterMark_ = highWaterMark;
449:        uint256 shareValue = convertToAssets(1e18);
450:        uint256 performanceFee = fees.performance;

452:        return
453:            performanceFee > 0 && shareValue > highWaterMark
454:                ? performanceFee.mulDiv(
455:                    (shareValue - highWaterMark) * totalSupply(),
456:                    1e36,
457:                    Math.Rounding.Down
458:                )
459:                : 0;
460:    }
```

```diff
diff --git a/src/vault/Vault.sol b/src/vault/Vault.sol
index 7a8f941..a977451 100644
--- a/src/vault/Vault.sol
+++ b/src/vault/Vault.sol
@@ -450,9 +450,9 @@ contract Vault is
         uint256 performanceFee = fees.performance;

         return
-            performanceFee > 0 && shareValue > highWaterMark
+            performanceFee > 0 && shareValue > highWaterMark_
                 ? performanceFee.mulDiv(
-                    (shareValue - highWaterMark) * totalSupply(),
+                    (shareValue - highWaterMark_) * totalSupply(),
                     1e36,
                     Math.Rounding.Down
                 )
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L57-L98
### Vault.sol.initialize(): asset has been cached already and should be used here(Save 1 SLOAD: ~100 gas)
```solidity
File: /src/vault/Vault.sol
57:    function initialize(

77:        asset = asset_; //@audit: asset cached here

80:        asset.approve(address(adapter_), type(uint256).max);//@audit: use cached value here
```

```diff
diff --git a/src/vault/Vault.sol b/src/vault/Vault.sol
index 7a8f941..0addeb4 100644
--- a/src/vault/Vault.sol
+++ b/src/vault/Vault.sol
@@ -77,7 +77,7 @@ contract Vault is
         asset = asset_;
         adapter = adapter_;

-        asset.approve(address(adapter_), type(uint256).max);
+        asset_.approve(address(adapter_), type(uint256).max);

```


## Internal/Private functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

Affected code:

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L89
```solidity
File: /src/vault/adapter/yearn/YearnAdapter.sol
89:    function _shareValue(uint256 yShares) internal view returns (uint256) {

101:    function _freeFunds() internal view returns (uint256) {

109:    function _yTotalAssets() internal view returns (uint256) {

114:    function _calculateLockedProfit() internal view returns (uint256) {
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L120-L123
```solidity
File: /src/vault/VaultController.sol
120:  function _deployVault(VaultInitParams memory vaultData, IDeploymentController _deploymentController)
121:    internal
122:    returns (address vault)
123:  {

141:  function _registerCreatedVault(
142:    address vault,
143:    address staking,
144:    VaultMetadata memory metadata
145:  ) internal {

154:  function _handleVaultStakingRewards(address vault, bytes memory rewardsData) internal {

225:  function __deployAdapter(
226:    DeploymentArgs memory adapterData,
227:    bytes memory baseAdapterData,
228:    IDeploymentController _deploymentController
229:  ) internal returns (address adapter) {

242:  function _encodeAdapterData(DeploymentArgs memory adapterData, bytes memory baseAdapterData)
243:    internal
244:    returns (bytes memory)
245:  {

256:  function _deployStrategy(DeploymentArgs memory strategyData, IDeploymentController _deploymentController)
257:    internal
258:    returns (address strategy)
259:  {

390:  function _registerVault(address vault, VaultMetadata memory metadata) internal {

692:  function _verifyAdapterConfiguration(address adapter, bytes32 adapterId) internal view {
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L191-L196
```solidity
File: /src/utils/MultiRewardStaking.sol
191:  function _lockToken(
192:    address user,
193:    IERC20 rewardToken,
194:    uint256 rewardAmount,
195:    EscrowInfo memory escrowInfo
196:  ) internal {
```

## Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time.
**Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it**

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. 
As an example, instead of repeatedly calling ```someMap[someIndex]```, save its reference like this: ```SomeStruct storage someStruct = someMap[someIndex]``` and use it.

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L52-L59
### TemplateRegistry.sol.addTemplateCategory(): templateCategoryExists\[templateCategory] should be cached in local storage
```solidity
File: /src/vault/TemplateRegistry.sol
52:  function addTemplateCategory(bytes32 templateCategory) external onlyOwner {
53:    if (templateCategoryExists[templateCategory]) revert TemplateCategoryExists(templateCategory);

55:    templateCategoryExists[templateCategory] = true;
56:    templateCategories.push(templateCategory);

58:    emit TemplateCategoryAdded(templateCategory);
59:  }
```


https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L402-L410
### MultiRewardStaking.sol.\_accrueRewards(): rewardInfos\[\_rewardToken] should be cached in local storage
```solidity
File: /src/utils/MultiRewardStaking.sol
402:  function _accrueRewards(IERC20 _rewardToken, uint256 accrued) internal {
403:    uint256 supplyTokens = totalSupply();
404:    uint224 deltaIndex;
405:    if (supplyTokens != 0)
406:      deltaIndex = accrued.mulDiv(uint256(10**decimals()), supplyTokens, Math.Rounding.Down).safeCastTo224();

408:    rewardInfos[_rewardToken].index += deltaIndex;
409:    rewardInfos[_rewardToken].lastUpdatedTimestamp = block.timestamp.safeCastTo32();
410:  }
```

## Emitting storage values instead of the memory one.(Save ~200 gas)
Here, the values emitted shouldn’t be read from storage. The existing memory values should be used instead:

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L629-L636
### Vault.sol.setQuitPeriod(): emit `_quitPeriod` : saves 1 SLOAD: ~100 gas
```solidity
File: /src/vault/Vault.sol
629:    function setQuitPeriod(uint256 _quitPeriod) external onlyOwner {
630:       if (_quitPeriod < 1 days || _quitPeriod > 7 days)
631:            revert InvalidQuitPeriod();

633:           quitPeriod = _quitPeriod;

635:        emit QuitPeriodSet(quitPeriod);
636:    }
```

```diff
diff --git a/src/vault/Vault.sol b/src/vault/Vault.sol
index 7a8f941..302ba9f 100644
--- a/src/vault/Vault.sol
+++ b/src/vault/Vault.sol
@@ -632,7 +632,7 @@ contract Vault is

         quitPeriod = _quitPeriod;

-        emit QuitPeriodSet(quitPeriod);
+        emit QuitPeriodSet(_quitPeriod);
     }

```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L57-L98
### Vault.sol.initialize(): asset has been cached already and cached value should be emitted here(Save 1 SLOAD : ~100 gas)

```solidity
File: /src/vault/Vault.sol
57:    function initialize(

77:        asset = asset_; //@audit: asset cached here

97:        emit VaultInitialized(contractName, address(asset));
```

```diff
diff --git a/src/vault/Vault.sol b/src/vault/Vault.sol
index 7a8f941..8562995 100644
--- a/src/vault/Vault.sol
+++ b/src/vault/Vault.sol
@@ -94,7 +94,7 @@ contract Vault is
             abi.encodePacked("Popcorn", name(), block.timestamp, "Vault")
         );

-        emit VaultInitialized(contractName, address(asset));
+        emit VaultInitialized(contractName, address(asset_));
     }
```

## Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L33-L35
```solidity
File: /src/vault/TemplateRegistry.sol
33:  mapping(bytes32 => bool) public templateExists;

35:  mapping(bytes32 => bool) public templateCategoryExists;
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L216-L218
```solidity
File: /src/utils/MultiRewardStaking.sol
216:  mapping(address => mapping(IERC20 => uint256)) public userIndex;

218:  mapping(address => mapping(IERC20 => uint256)) public accruedRewards;
```

## Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L157
```solidity
File: /src/utils/MultiRewardEscrow.sol
157:      Escrow memory escrow = escrows[escrowId];
```


```diff
diff --git a/src/utils/MultiRewardEscrow.sol b/src/utils/MultiRewardEscrow.sol
index cf50b08..6ee7fe9 100644
--- a/src/utils/MultiRewardEscrow.sol
+++ b/src/utils/MultiRewardEscrow.sol
@@ -154,13 +154,13 @@ contract MultiRewardEscrow is Owned {
   function claimRewards(bytes32[] memory escrowIds) external {
     for (uint256 i = 0; i < escrowIds.length; i++) {
       bytes32 escrowId = escrowIds[i];
-      Escrow memory escrow = escrows[escrowId];
+      Escrow storage escrow = escrows[escrowId];

       uint256 claimable = _getClaimableAmount(escrow);
       if (claimable == 0) revert NotClaimable(escrowId);

-      escrows[escrowId].balance -= claimable;
-      escrows[escrowId].lastUpdateTime = block.timestamp.safeCastTo32();
+      escrow.balance -= claimable;
+      escrow.lastUpdateTime = block.timestamp.safeCastTo32();

       escrow.token.safeTransfer(escrow.account, claimable);
       emit RewardsClaimed(escrow.token, escrow.account, claimable);
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L254
```solidity
File:  /src/utils/MultiRewardStaking.sol   
254: RewardInfo memory rewards = rewardInfos[rewardToken];
```

As rewards now points to the storage variable we can just use it to effect changes to the storage one, this way we  no longer need to write to `rewardInfos[rewardToken];` at the end as writing to `rewards` would have a similar impact
```diff
diff --git a/src/utils/MultiRewardStaking.sol b/src/utils/MultiRewardStaking.sol
index 95ebefd..2cf3289 100644
--- a/src/utils/MultiRewardStaking.sol
+++ b/src/utils/MultiRewardStaking.sol
@@ -251,7 +251,7 @@ contract MultiRewardStaking is ERC4626Upgradeable, OwnedUpgradeable {
   ) external onlyOwner {
     if (asset() == address(rewardToken)) revert RewardTokenCantBeStakingToken();

-    RewardInfo memory rewards = rewardInfos[rewardToken];
+    RewardInfo storage rewards = rewardInfos[rewardToken];
     if (rewards.lastUpdatedTimestamp > 0) revert RewardTokenAlreadyExist(rewardToken);

     if (amount > 0) {
@@ -276,7 +276,7 @@ contract MultiRewardStaking is ERC4626Upgradeable, OwnedUpgradeable {
       ? block.timestamp.safeCastTo32()
       : _calcRewardsEnd(0, rewardsPerSecond, amount);

-    rewardInfos[rewardToken] = RewardInfo({
+    rewards = RewardInfo({
       ONE: ONE,
       rewardsPerSecond: rewardsPerSecond,
       rewardsEndTimestamp: rewardsEndTimestamp,
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L297
```solidity
File: /src/utils/MultiRewardStaking.sol
297:    RewardInfo memory rewards = rewardInfos[rewardToken];
```
Note the changes done at the last lines. We no longer need to asign the variable at the end as we are not dealing with a copy of the variable but a reference to the storage variable, thus any changes to our locally cached storage variable would change the actual variable stored in storage

```diff
diff --git a/src/utils/MultiRewardStaking.sol b/src/utils/MultiRewardStaking.sol
index 95ebefd..b0ce54a 100644
--- a/src/utils/MultiRewardStaking.sol
+++ b/src/utils/MultiRewardStaking.sol
@@ -294,7 +294,7 @@ contract MultiRewardStaking is ERC4626Upgradeable, OwnedUpgradeable {
    * @dev The `rewardsEndTimestamp` gets calculated based on `rewardsPerSecond` and `amount`.
    */
   function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {
-    RewardInfo memory rewards = rewardInfos[rewardToken];
+    RewardInfo storage rewards = rewardInfos[rewardToken];

     if (rewardsPerSecond == 0) revert ZeroAmount();
     if (rewards.lastUpdatedTimestamp == 0) revert RewardTokenDoesntExist(rewardToken);
@@ -310,8 +310,8 @@ contract MultiRewardStaking is ERC4626Upgradeable, OwnedUpgradeable {
       rewardsPerSecond,
       remainder
     );
-    rewardInfos[rewardToken].rewardsPerSecond = rewardsPerSecond;
-    rewardInfos[rewardToken].rewardsEndTimestamp = rewardsEndTimestamp;
+    rewards.rewardsPerSecond = rewardsPerSecond;
+    rewards.rewardsEndTimestamp = rewardsEndTimestamp;
   }
```


https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L328
```solidity
File: /src/utils/MultiRewardStaking.sol
328:    RewardInfo memory rewards = rewardInfos[rewardToken];
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L414
```solidity
File: /src/utils/MultiRewardStaking.sol
414:    RewardInfo memory rewards = rewardInfos[_rewardToken];
```

##  IF's/require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L243-L288
### cheaper to reorder the if's check to have the cheap check for functional parameter before other operations
```solidity
File: /src/utils/MultiRewardStaking.sol
243:  function addRewardToken(
244:    IERC20 rewardToken,
245:    uint160 rewardsPerSecond,
246:    uint256 amount,
247:    bool useEscrow,
248:    uint192 escrowPercentage,
249:    uint32 escrowDuration,
250:    uint32 offset
251:  ) external onlyOwner {
252:    if (asset() == address(rewardToken)) revert RewardTokenCantBeStakingToken();

254:    RewardInfo memory rewards = rewardInfos[rewardToken];
255:    if (rewards.lastUpdatedTimestamp > 0) revert RewardTokenAlreadyExist(rewardToken);

257:    if (amount > 0) {
258:      if (rewardsPerSecond == 0) revert ZeroRewardsSpeed();
259:      rewardToken.safeTransferFrom(msg.sender, address(this), amount);
260:    }
```

The first check involves a  functional call which is expensive. As we also revert on a functional parameter (rewardsPerSecond) failure to pass a certain check (in this case , we revert if it's equal to zero) it would be way cheaper to reorder the if's cheak to have the cheap check for functional parameter before performing the more expensive ones.

```diff
diff --git a/src/utils/MultiRewardStaking.sol b/src/utils/MultiRewardStaking.sol
index 95ebefd..b1723a0 100644
--- a/src/utils/MultiRewardStaking.sol
+++ b/src/utils/MultiRewardStaking.sol
@@ -249,13 +249,13 @@ contract MultiRewardStaking is ERC4626Upgradeable, OwnedUpgradeable {
     uint32 escrowDuration,
     uint32 offset
   ) external onlyOwner {
+    if (rewardsPerSecond == 0) revert ZeroRewardsSpeed();
     if (asset() == address(rewardToken)) revert RewardTokenCantBeStakingToken();

     RewardInfo memory rewards = rewardInfos[rewardToken];
     if (rewards.lastUpdatedTimestamp > 0) revert RewardTokenAlreadyExist(rewardToken);

     if (amount > 0) {
-      if (rewardsPerSecond == 0) revert ZeroRewardsSpeed();
       rewardToken.safeTransferFrom(msg.sender, address(this), amount);
     }
```


https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L296-L303
### Move the functional parameter check to be the first operation
```solidity
File: /src/utils/MultiRewardStaking.sol
296:  function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {
297:    RewardInfo memory rewards = rewardInfos[rewardToken];

299:    if (rewardsPerSecond == 0) revert ZeroAmount();
300:    if (rewards.lastUpdatedTimestamp == 0) revert RewardTokenDoesntExist(rewardToken);
301:    if (rewards.rewardsPerSecond == 0) revert RewardsAreDynamic(rewardToken);

303:    _accrueRewards(rewardToken, _accrueStatic(rewards));
```

As we have a check for a  functional parameter, it is better to do that check first before anything else. This way we don't waste any gas if the function parameter does not the required condition . In the above function we shouldn't waste gas on `RewardInfo memory rewards = rewardInfos[rewardToken];` if we could end up reverting on ` if (rewardsPerSecond == 0) revert ZeroAmount();`

```diff
diff --git a/src/utils/MultiRewardStaking.sol b/src/utils/MultiRewardStaking.sol
index 95ebefd..bb98e33 100644
--- a/src/utils/MultiRewardStaking.sol
+++ b/src/utils/MultiRewardStaking.sol
@@ -294,9 +294,10 @@ contract MultiRewardStaking is ERC4626Upgradeable, OwnedUpgradeable {
    * @dev The `rewardsEndTimestamp` gets calculated based on `rewardsPerSecond` and `amount`.
    */
   function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {
+    if (rewardsPerSecond == 0) revert ZeroAmount();
+
     RewardInfo memory rewards = rewardInfos[rewardToken];

-    if (rewardsPerSecond == 0) revert ZeroAmount();
     if (rewards.lastUpdatedTimestamp == 0) revert RewardTokenDoesntExist(rewardToken);
     if (rewards.rewardsPerSecond == 0) revert RewardsAreDynamic(rewardToken);
```


https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L57-L98
### Move the if for functional parameter to the top
```solidity
File: /src/vault/Vault.sol
57:    function initialize(
58:        IERC20 asset_,
59:        IERC4626 adapter_,
60:        VaultFees calldata fees_,
61:        address feeRecipient_,
62:        address owner
63:    ) external initializer {
64:        __ERC20_init(
65:            string.concat(
66:                "Popcorn ",
67:                IERC20Metadata(address(asset_)).name(),
68:                " Vault"
69:            ),
70:            string.concat("pop-", IERC20Metadata(address(asset_)).symbol())
71:        );
72:        __Owned_init(owner);

74:        if (address(asset_) == address(0)) revert InvalidAsset();
75:        if (address(asset_) != adapter_.asset()) revert InvalidAdapter();

77:        asset = asset_;
78:        adapter = adapter_;

80:        asset.approve(address(adapter_), type(uint256).max);

82:        _decimals = IERC20Metadata(address(asset_)).decimals();

84:        INITIAL_CHAIN_ID = block.chainid;
85:        INITIAL_DOMAIN_SEPARATOR = computeDomainSeparator();

87:        feesUpdatedAt = block.timestamp;
88:        fees = fees_;

90:        if (feeRecipient_ == address(0)) revert InvalidFeeRecipient();
```
As `feeRecipient_` is a functional parameter, we should check it first before performing other operations to prevent wasting gas if we will ultimately revert due to `feeRecipient_` not passing the required checks.

```diff
diff --git a/src/vault/Vault.sol b/src/vault/Vault.sol
index 7a8f941..10d6b1b 100644
--- a/src/vault/Vault.sol
+++ b/src/vault/Vault.sol
@@ -71,6 +71,7 @@ contract Vault is
         );
         __Owned_init(owner);

+        if (feeRecipient_ == address(0)) revert InvalidFeeRecipient();
         if (address(asset_) == address(0)) revert InvalidAsset();
         if (address(asset_) != adapter_.asset()) revert InvalidAdapter();

@@ -87,7 +88,6 @@ contract Vault is
         feesUpdatedAt = block.timestamp;
         fees = fees_;

-        if (feeRecipient_ == address(0)) revert InvalidFeeRecipient();
         feeRecipient = feeRecipient_;

         contractName = keccak256(
```

## `keccak256()` should only need to be called on a specific string literal once

It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to `bytes4` should also only be done once

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L466
```solidity
File: /src/utils/MultiRewardStaking.sol
466:      keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L495
```solidity
File: /src/utils/MultiRewardStaking.sol
495:          keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L497
```solidity
File: /src/utils/MultiRewardStaking.sol
497:
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L685-L687
```solidity
File: /src/vault/Vault.sol
685:        keccak256(
686:            "Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"
687:        ),
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L720-L722
```solidity
File: /src/vault/Vault.sol
720:        keccak256(
721:            "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"
722:        ),
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L724
```solidity
File: /src/vault/Vault.sol
724:      keccak256("1"),
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L653-L655
```solidity
File: /src/vault/adapter/abstracts/AdapterBase.sol
653:        keccak256(
654:            "Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"
655:        ),

688:        keccak256(
689:                "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"
690:            ),

692:         keccak256("1"),
```


## x += y costs more gas than x = x + y for state variables
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L158
```solidity
File: /src/vault/adapter/abstracts/AdapterBase.sol
158:        underlyingBalance += _underlyingBalance() - underlyingBalance_;
```

```diff
-        underlyingBalance += _underlyingBalance() - underlyingBalance_;
+        underlyingBalance = underlyingBalance + _underlyingBalance() - underlyingBalance_;
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L225
```solidity
File: /src/vault/adapter/abstracts/AdapterBase.sol
225:            underlyingBalance -= underlyingBalance_ - _underlyingBalance();
```

## Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Use a larger size then downcast where needed

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L88-L94
```solidity
File: /src/utils/MultiRewardEscrow.sol

//@audit: uint32 duration, uint32 offset
88:  function lock(
89:    IERC20 token,
90:    address account,
91:    uint256 amount,
92:    uint32 duration,
93:    uint32 offset
94:  ) external {
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L243-L251
```solidity
File: /src/utils/MultiRewardStaking.sol

//@audit: uint160 rewardsPerSecond,uint192 escrowPercentage,uint32 escrowDuration, uint32 offset
243:  function addRewardToken(
244:    IERC20 rewardToken,
245:    uint160 rewardsPerSecond,
246:    uint256 amount,
247:    bool useEscrow,
248:    uint192 escrowPercentage,
249:    uint32 escrowDuration,
250:    uint32 offset
251:  ) external onlyOwner {

//@audit: uint160 rewardsPerSecond
296:  function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {

//@audit: uint32 rewardsEndTimestamp, uint160 rewardsPerSecond,
351:  function _calcRewardsEnd(
352:    uint32 rewardsEndTimestamp,
353:    uint160 rewardsPerSecond,
354:    uint256 amount
355:  ) internal returns (uint32) {
```

## Using unchecked blocks to save gas
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
[see resource](https://github.com/ethereum/solidity/issues/10695)

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L151
```solidity
File: /src/vault/adapter/yearn/YearnAdapter.sol
151:        return _depositLimit - assets;
```
The operation `_depositLimit - assets` cannot underflow due to the check on [Line 150](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L150) that ensures that `_depositLimit` is greater than `assets` before performing the arithmetic operation

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L357
```solidity
File: /src/utils/MultiRewardStaking.sol
357:      amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);
```
The operation `rewardsEndTimestamp - block.timestamp` cannot underflow due to the check on [Line 356](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L356) that ensures that `rewardsEndTimestamp` is greater than `block.timestamp`


https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L395
```solidity
File: /src/utils/MultiRewardStaking.sol
395:      elapsed = rewards.rewardsEndTimestamp - rewards.lastUpdatedTimestamp;
```

The operation `rewards.rewardsEndTimestamp - rewards.lastUpdatedTimestamp` cannot underflow due to the check on [Line 394](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L394) that ensures that `rewards.rewardsEndTimestamp` is greater than `rewards.lastUpdatedTimestamp` before perfoming the arithmetic operation

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L455
```solidity
File: /src/vault/Vault.sol
455:                    (shareValue - highWaterMark) * totalSupply(),
```

The operation `shareValue - highWaterMark` cannot underflow as this operation would only be performed if `shareValue` is greater than `highWaterMark` due to the condition check on [Line 453](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L453)

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L537
```solidity
File: /src/vault/adapter/abstracts/AdapterBase.sol
537:                    (shareValue - highWaterMark_) * totalSupply(),
```
The operation `shareValue - highWaterMark_` cannot underflow due to the check on [Line 535](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L535) that ensures that `shareValue` is greater than `highWaterMark_` before performing the arithmetic operation

## Using unchecked blocks to save gas - Increments in for loop can be unchecked  ( save 30-40 gas per loop iteration)
The majority of Solidity for loops increment a uint256 variable that starts at 0. These increment operations never need to be checked for over/underflow because the variable will never reach the max number of uint256 (will run out of gas long before that happens). The default over/underflow check wastes gas in every iteration of virtually every for loop.


**Affected code**
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L42
```solidity
File: /src/vault/PermissionRegistry.sol
42:    for (uint256 i = 0; i < len; i++) {
```
The above should be modified to:
```diff
diff --git a/src/vault/PermissionRegistry.sol b/src/vault/PermissionRegistry.sol
index 1a61e1c..fa1c7bb 100644
--- a/src/vault/PermissionRegistry.sol
+++ b/src/vault/PermissionRegistry.sol
@@ -39,12 +39,15 @@ contract PermissionRegistry is Owned {
     uint256 len = targets.length;
     if (len != newPermissions.length) revert Mismatch();

-    for (uint256 i = 0; i < len; i++) {
+    for (uint256 i = 0; i < len;) {
       if (newPermissions[i].endorsed && newPermissions[i].rejected) revert Mismatch();

       emit PermissionSet(targets[i], newPermissions[i].endorsed, newPermissions[i].rejected);

       permissions[targets[i]] = newPermissions[i];
+      unchecked {
+        ++i;
+      }
     }
```

**Other Instances to modify**

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L53
```solidity
File: /src/utils/MultiRewardEscrow.sol
53:    for (uint256 i = 0; i < escrowIds.length; i++) {

155:    for (uint256 i = 0; i < escrowIds.length; i++) {

210:    for (uint256 i = 0; i < tokens.length; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L171
```solidity
File: /src/utils/MultiRewardStaking.sol
171:    for (uint8 i; i < _rewardTokens.length; i++) {

373:    for (uint8 i; i < _rewardTokens.length; i++) {
```

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L319
```solidity
File: /src/vault/VaultController.sol
319:    for (uint8 i = 0; i < len; i++) {

337:    for (uint8 i = 0; i < len; i++) {

357:    for (uint8 i = 0; i < len; i++) {

374:    for (uint8 i = 0; i < len; i++) {

437:    for (uint256 i = 0; i < len; i++) {

494:    for (uint256 i = 0; i < len; i++) {

523:    for (uint256 i = 0; i < len; i++) {

564:    for (uint256 i = 0; i < len; i++) {

587:    for (uint256 i = 0; i < len; i++) {

607:    for (uint256 i = 0; i < len; i++) {

620:    for (uint256 i = 0; i < len; i++) {

633:    for (uint256 i = 0; i < len; i++) {

646:    for (uint256 i = 0; i < len; i++) {

766:    for (uint256 i = 0; i < len; i++) {

806:    for (uint256 i = 0; i < len; i++) {
```
[see resource](https://github.com/ethereum/solidity/issues/10695)


