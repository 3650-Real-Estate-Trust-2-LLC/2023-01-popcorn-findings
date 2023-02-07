# Gas Optimizations Report for Popcorn contest
## Overview

During the audit, 4 gas issues were found.  
Total savings ~7145.

№ | Title | Instance Count | Saved
--- | --- | --- | ---
G-1 | Use unchecked blocks for incrementing | 22 | 770
G-2 | Use unchecked blocks for subtractions where underflow is impossible | 5 | 175
G-3 | Cached variable is not used  | 1 | 200
G-4 | Using ```storage``` pointer to ```struct```/```array```/```mapping``` is cheaper than using ```memory``` | 8 | 6000

## Gas Optimizations Findings(4)
### G-1. Use unchecked blocks for incrementing
##### Description
In Solidity 0.8+, there’s a default overflow and underflow check on unsigned integers. In the loops or similar cases, "i" will not overflow because the loop will run out of gas before that or max value never be reached.
##### Instances
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L42
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L53
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L102
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L155
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L210
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L171
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L373
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L319
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L337
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L357 
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L374
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L437
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L494
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L523
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L564
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L587
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L607
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L620
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L633
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L646
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L766
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L806

##### Recommendation
Change:
```
for (uint256 i; i < n; ++i) {
 // ...
}
```
to:
```
for (uint256 i; i < n;) { 
 // ...
 unchecked { ++i; }
}
```

##### Saved
This saves ~30-40 gas per iteration.  
So, ~35*22 = 770
#
### G-2. Use unchecked blocks for subtractions where underflow is impossible
##### Description
In Solidity 0.8+, there’s a default overflow and underflow check on unsigned integers. When an overflow or underflow isn’t possible (after require or if-statement), some gas can be saved by using [unchecked blocks](https://docs.soliditylang.org/en/v0.8.17/control-structures.html#checked-or-unchecked-arithmetic).
##### Instances
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L151
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L357 (rewardsEndTimestamp - block.timestamp)
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L395
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L455 (shareValue - highWaterMark)
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L537 (shareValue - highWaterMark_)

##### Saved
This saves ~35.  
So, ~35*5 = 175
#
### G-3. Cached variable is not used
##### Description
The variable ``` highWaterMark_``` was meant to be used, but ```highWaterMark``` was used.
##### Instances
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L448
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L453
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L455

##### Saved
This saves 100*2 = 200.
#
### G-4. Using ```storage``` pointer to ```struct```/```array```/```mapping``` is cheaper than using ```memory```
##### Description
When you copy a storage ```struct```/```array```/```mapping``` to a memory variable, you are copying each member by reading it from the storage, which is expensive, but when you use the ```storage``` keyword, you are just storing a pointer to the storage, which is much cheaper.
##### Instances
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L157
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L176 
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L254
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L297
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L328
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L372
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L375
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L414

##### Recommendation
For example, change: 
```
Escrow memory escrow = escrows[escrowId];
```  
to:  
```
Escrow storage escrow = escrows[escrowId];
```

##### Saved
This saves ~6000.  