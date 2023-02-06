  # 1. [G-1] Arithmetics: uncheck blocks for arithmetics operations that can't underflow/overflow

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn't possible, some gas can be saved by using an `unchecked` block.

I suggest wrapping with an `unchecked` block here:

    File src/vault/adapter/yearn/YearnAdapter.sol, line 151:     return _depositLimit - assets;

    File src/utils/MultiRewardEscrow.sol, line 102:     nonce++;
    File src/utils/MultiRewardEscrow.sol, line 110:     amount -= fee;
    File src/utils/MultiRewardEscrow.sol, line 162:     escrows[escrowId].balance -= claimable;

    File src/utils/MultiRewardStaking.sol, line 424:     uint256 deltaIndex = rewards.index - oldIndex;
    File src/utils/MultiRewardStaking.sol, line 431:     accruedRewards[_user][_rewardToken] += supplierDelta;

    File src/vault/Vault.sol, line 228:     shares += feeShares;

# 2. [G-2] For loops: increments in for loop can be uncheck to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block

    File src/utils/MultiRewardEscrow.sol, line 53, 155, 210.

    File src/utils/MultiRewardStaking.sol, line 171, 373.

    File src/vault/PermissionRegistry.sol, line 42.

    File src/vault/VaultController.soll, line 319, 337, 357, 374, 437, 494, 523, 564, 587, 607, 620, 633, 646, 766, 806.

The code would go from:

    for (uint256 i; i < numIterations; i++) { 
      ...
    }

to:

    for (uint256 i; i < numIterations;) { 
      ...
      unchecked { ++i; }  
    }

# 3. [G-03] Uncheck `transfer` in *src/utils/MultiRewardStaking.sol*

Boolean return value for `transfer` is not checked in file src/utils/MultiRewardStaking.sol line 182:

    _rewardTokens[i].transfer(user, rewardAmount); // I recommend:  require(_rewardTokens[i].transfer(user, rewardAmount);, "!transferReward");

# 4. [G-4] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES

There are 6 instances of this issue:

    File src/utils/MultiRewardStaking.sol, line 357, 408, 431.
    File src/vault/Vault.sol, line 228, 343, 365.

# 5. [G-5] Can make variable outside the loop to save gas

Make it outside and only use it inside.

There are 19 instances of this issue:

    File src/utils/MultiRewardEscrow.sol, line 157, 159.

    File src/utils/MultiRewardStaking.sol, line 176, 375.

    File src/vault/VaultController.sol, line 323, 338, 360, 375, 439, 450, 497, 565, 588, 609, 620, 635, 648, 767, 807.
