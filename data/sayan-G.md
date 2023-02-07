## [G-01] USAGE OF UINTS/INTS SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD
> When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html)

Each operation involving a  `uint8`  costs an extra  [**22-28 gas**](https://gist.github.com/IllIllI000/9388d20c70f9a4632eb3ca7836f54977)  (depending on whether the other operand is also a variable of type  `uint8`) as compared to ones involving  `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the  `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.
```
File: src/vault/strategy/StrategyBase.sol
13:     uint8 len = uint8(sigs.length);

File: src/vault/adapter/abstracts/AdapterBase.sol
38:     uint8 internal _decimals;

File: src/vault/adapter/abstracts/AdapterBase.sol
637:         uint8 v,

File: src/vault/VaultController.sol
314:     uint8 len = uint8(vaults.length);

File: src/vault/Vault.sol
38:     uint8 internal _decimals;

File: src/vault/Vault.sol
669:         uint8 v,

File: src/utils/MultiRewardStaking.sol
33:   uint8 private _decimals;
```


## [G-02] += COSTS MORE GAS THAN = + FOR STATE VARIABLES
Use = + or = - instead to save gas
```
File: src/vault/Vault.sol
228:         shares += feeShares;

File: src/vault/Vault.sol
343:         shares += shares.mulDiv(

File: src/vault/Vault.sol
365:         assets += assets.mulDiv(

File: src/vault/adapter/abstracts/AdapterBase.sol
158:         underlyingBalance += _underlyingBalance() - underlyingBalance_;

File: src/utils/MultiRewardStaking.sol
357:       amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);

File: src/utils/MultiRewardStaking.sol
408:     rewardInfos[_rewardToken].index += deltaIndex;

File: src/utils/MultiRewardStaking.sol
431:     accruedRewards[_user][_rewardToken] += supplierDelta;
```
## [G-03]USE FUNCTION INSTEAD OF MODIFIERS
use function instead of modifiers like this
```
with modifer:
 modifier onlyOwner { 
        require(msg.sender == owner, "Smart Account:: Sender is not authorized");
         _; 
        }
        
with function:
	// modifier onlyOwner { 
    function onlyOwner() private view{
         require(msg.sender == owner, "Smart Account:: Sender is not authorized");
          // _; 
        }
```
Then place onlyOwner() in the first line of the functions in which modifier was used 

```
File: src/vault/Vault.sol
480:     modifier takeFees() {
481:         uint256 managementFee = accruedManagementFee();
482:         uint256 totalFee = managementFee + accruedPerformanceFee();
483:         uint256 currentAssets = totalAssets();
484:         uint256 shareValue = convertToAssets(1e18);
485: 
486:         if (shareValue > highWaterMark) highWaterMark = shareValue;
487: 
488:         if (managementFee > 0) feesUpdatedAt = block.timestamp;
489: 
490:         if (totalFee > 0 && currentAssets > 0)
491:             _mint(feeRecipient, convertToShares(totalFee));
492: 
493:         _;
494:     }


File: src/vault/Vault.sol
496:     modifier syncFeeCheckpoint() {
497:         _;
498:         highWaterMark = convertToAssets(1e18);
499:     }

File: src/vault/Vault.sol
496:     modifier syncFeeCheckpoint() {
497:         _;
498:         highWaterMark = convertToAssets(1e18);
499:     }

File: src/vault/VaultController.sol
704:   modifier canCreate() {
705:     if (
706:       permissionRegistry.endorsed(address(1))
707:         ? !permissionRegistry.endorsed(msg.sender)
708:         : permissionRegistry.rejected(msg.sender)
709:     ) revert NotAllowed(msg.sender);
710:     _;
711:   }

File: src/vault/adapter/abstracts/AdapterBase.sol
559:     modifier takeFees() {
560:         _;
561: 
562:         uint256 fee = accruedPerformanceFee();
563:         if (fee > 0) _mint(FEE_RECIPIENT, convertToShares(fee));
564: 
565:         uint256 shareValue = convertToAssets(1e18);
566:         if (shareValue > highWaterMark) highWaterMark = shareValue;
567:     }

File: src/vault/adapter/abstracts/OnlyStrategy.sol
10:   modifier onlyStrategy() {
11:     if (msg.sender != address(this)) revert NotStrategy(msg.sender);
12:     _;
13:   }


```