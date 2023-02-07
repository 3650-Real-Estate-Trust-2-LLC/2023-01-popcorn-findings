**Description**:

Use `safeTransferFrom` instead of `transferFrom`

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L526

```solidity
      rewardTokens[i].transferFrom(msg.sender, address(this), amounts[i]);
```