# 1. Using `storage` instead of `memory` for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from `storage`, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new `memory` variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct.

Affected line of code:

- [MultiRewardEscrow.sol#L157](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L157)
- [MultiRewardStaking.sol#L176](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L176)
- [MultiRewardStaking.sol#L357](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L357)
- [MultiRewardStaking.sol#L408](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L408)
- [MultiRewardStaking.sol#L431](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L431)

# 2. Regular addition/subtraction assignment saves gas

Using standard addition or subtraction assignment (`x = x + y` or `x = x - y`) rather than the shorthand versions (`x += y` or `x -= y`) saves gas. This can save approximately 22 gas per occurrence.

For example on [line 162](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L162) may be refactored as:

```solidity
File: /src/Drips.sol
Line 162:    escrows[escrowId].balance = escrows[escrowId].balance - claimable;
```

Affected line of code:

- [MultiRewardEscrow.sol#L110](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L110)

- [MultiRewardEscrow.sol#L162](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L162)

# 3. Uncheck arithmetics operations that can’t underflow/overflow

When an overflow or an underflow isn’t possible, some gas can be saved using an unchecked block.

For example on the [lines 155-168](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L155-L168) you would refactor it like so:

```diff
File: /src/Drips.sol
+    for (uint256 i = 0; i < escrowIds.length;)
-    for (uint256 i = 0; i < escrowIds.length; i++) {
      bytes32 escrowId = escrowIds[i];
      Escrow memory escrow = escrows[escrowId];


      uint256 claimable = _getClaimableAmount(escrow);
      if (claimable == 0) revert NotClaimable(escrowId);


      escrows[escrowId].balance -= claimable;
      escrows[escrowId].lastUpdateTime = block.timestamp.safeCastTo32();


      escrow.token.safeTransfer(escrow.account, claimable);
      emit RewardsClaimed(escrow.token, escrow.account, claimable);

+     unchecked { i++ }
    }




```

Affected line of code:

- [MultiRewardEscrow.sol#L53](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L53)
- [MultiRewardEscrow.sol#L155](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L155)
- [MultiRewardEscrow.sol#L210](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L210)
- [VaultController.sol#L319](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L319)
- [VaultController.sol#L337](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L337)
- [VaultController.sol#L357](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L357)
- [VaultController.sol#L374](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L374)
- [VaultController.sol#L437](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L437)
- [VaultController.sol#L494](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L494)
- [VaultController.sol#L523](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L523)
- [VaultController.sol#L564](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L564)
- [VaultController.sol#L587](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L587)
- [VaultController.sol#L607](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L607)
- [VaultController.sol#L620](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L620)
- [VaultController.sol#L633](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L633)
- [VaultController.sol#L646](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L646)
- [VaultController.sol#L766](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L766)
- [VaultController.sol#L806](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L806)
