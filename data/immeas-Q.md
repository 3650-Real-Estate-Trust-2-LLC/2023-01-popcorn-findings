# low

## L-1 no invalidate nonce

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L627

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L659

If a user wants to withdraw their permit there's no way currently.

## L-2 no validation on fee parameters

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L88

There's validation when changing them but not creating the vault. possibly you could be stuck with bad fee configuration for three days

## L-3 `Vault::withdraw` triggers `syncFeeCheckpoint` but not `Vault::redeem`

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L257

Either none of them should trigger or both of them.

## non-crit

## NC-1 unused function parameter

`vault` is not used

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L390

remove if not used

## NC-2 unused variable

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L448

the state variable `highWaterMark` is mistakenly used instead of `highWaterMark_` in the code snippet:

```javascript
File: Vault.sol

453:            performanceFee > 0 && shareValue > highWaterMark
454:                ? performanceFee.mulDiv(
455:                    (shareValue - highWaterMark) * totalSupply(),
456:                    1e36,
457:                    Math.Rounding.Down
458:                )
459:                : 0;
```

## NC-3 incorrect natspec

### 1 
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L60

adds a new template not new category and caller must not be owner at all

### 2
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L425

calculation is done based on seconds not minutes

### 3
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L181

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L274

caller is canCreate not owner


## NC-4 wrong error

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L98

`duration` is not an amount

## NC-5 consider returning `id` from `MultiRewardEscrow::lock`

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L88-L130

