# Report

## 01: `MultiRewardEscrow.isClaimable()` always returns true

```sol
  function isClaimable(bytes32 escrowId) external view returns (bool) {
    return escrows[escrowId].lastUpdateTime != 0 && escrows[escrowId].balance > 0;
  }
```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L141

When an escrow is created both `lastUpdateTime` and `balance` will be `!= 0`. But, that doesn't mean that the escrow is claimable, e.g. it might not have started yet.

## 02: Vault's fees are not validated on initialization

When new fees are proposed for a vault they are validated to be `<= 1e18`. That doesn't happen in the `initialize()` function. The vault creator could shoot themselves in the foot by setting the wrong fees.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L88
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L526

## 03: Vault's `previewMint()` and `previewWithdraw()` take more fees than specified

The calculation is wrong. Instead of `assets * fee / (1e18 - fee)` it should be `assets * fee / 1e18` to determine the fee.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L345
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L367

