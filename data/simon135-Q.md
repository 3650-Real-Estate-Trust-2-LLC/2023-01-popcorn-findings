##### `SECONDS_PER_YEAR` is wrong a year is 365 days 
The constant in the code is  `365.25 days` which is a year  and a little  bit more 
but a year is just 365 days
https://github.com/code-423n4/2023-01-popcorn/blob/7a513a9734b9e49af33041e2032ffc131f3b73b0/src/vault/Vault.sol#L35
#### when the creator sets fees there is no  check that fees are less than `1e18`
When a creator wants to change fees they have to set them to less than `1e18`.
```solidity
        if (
            newFees.deposit >= 1e18 ||
            newFees.withdrawal >= 1e18 ||
            newFees.management >= 1e18 ||
            newFees.performance >= 1e18
        ) revert InvalidVaultFees();

```
but on   the `initialize` function, there is such a check 
```solidity
        fees = fees_;
```
Make checks  for the `initialize` function

##### their is an extra `_verifyToken()` function in `deployAdapter()`
When `deployAdapter` is called in the `deployVault` function, the asset is verified 
so when `deployAdapter` is called the token is already verified
Remove the `verifyToken` from the inside call 
https://github.com/code-423n4/2023-01-popcorn/blob/7a513a9734b9e49af33041e2032ffc131f3b73b0/src/vault/VaultController.sol#L192

##### `targets` parameters do not represent   `AdminProxy
the `targets` instead are users  that get permission
```solidity
* @param targets `AdminProxy`

```
https://github.com/code-423n4/2023-01-popcorn/blob/7a513a9734b9e49af33041e2032ffc131f3b73b0/src/vault/VaultController.sol#L404
#####  have so check for `address(0)` and `address(1)`  in `setPermissions` function
In  the `setPermissions` function `0x` represents tokens and `0x1` represents users 
Make an `if` check that checks that input and set it accordingly
Just to help readability 
 ##### There is a discrepancy between `AdapaterBase->setHarvestCooldown`  and `VaultController->setHarvestCooldown` functions
  in AdapterBase the  function checks
```solidity 
if (newCooldown >= 1 days) revert InvalidHarvestCooldown(newCooldown);
```
in VaultController  the function checks 

```solidity 
    if (newCooldown > 1 days) revert InvalidHarvestCooldown(newCooldown);
```
 `vaultController->HarvestCooldown` is set on the adapter contract  which can `=`  1 day 
but in the `adapterBase`  the cooldown can't be  `=`1 day 
The worst thing that can happen  undesired cooldown
 

