# 1. VaultRegistry's getSubmitter function has wrong return value

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IVaultRegistry.sol#L27

The interface IVaultRegistry has defined `getSubmitter` to return an address.

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L75-L77

But this code is returning `VaultMetadata`, which is wrong. In this implementation, this function is working exactly same as getVault, which should not be expected.

Suggestion: delete this getSubmitter function

# 2. AdapterBase's FEE_RECIPIENT can not be changed

No change can be made for this FEE_RECIPIENT, consider adding a function to change it

# 3. Should not emit Harvested event if no harvest function is called

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L438-L450

Even when harvestCooldown check is not passed, this harvest function still emit the `Harvested` event.

# 4. `lastHarvest` is not changed in harvest function

This means `lastHarvest` must be changed in the delegate call of strategy contract, which is error-prone because it requires exact variable layout.

Suggestion: change to

```
    function harvest() public takeFees {
        if (
            address(strategy) != address(0) &&
            ((lastHarvest + harvestCooldown) < block.timestamp)
        ) {
            // solhint-disable
            address(strategy).delegatecall(
                abi.encodeWithSignature("harvest()")
            );
            lastHarvest = block.timestamp;
            emit Harvested();
        }

    }
```

----

# 5. VaultController is missing call to setQuitPeriod and  setFeeRecipient to Vault

There're 6 Vault functions that require `onlyOwner`: proposeFees setFeeRecipient proposeAdapter setQuitPeriod pause unpause

The owner of Vault is the AdminProxy, which gives VaultController permission to call admin functions.

For example, proposeFees is called by proposeVaultFees.

However, 2 functions are missing corresponding function:   setQuitPeriod and  setFeeRecipient

This makes nobody can change quit period  and fee recipient of Vaults.

Suggestion: add setVaultQuitPeriod  and setVaultFeeRecipient

# 6. Missing removeClone in CloneRegistry

There is no removeClone function in CloneRegistry, if we found any clone is vulnerable, we cannot remove it from the CloneRegistry, so that we can not prevent it from being proposed as adapters.

# 7. VaultController's deployStaking incompatible with created Vault

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L278-L281

`depoloyStaking` requires that asset is not a clone by calling _verifyToken; but `_deployStaking` function is also called at https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L108 for the new created vault, which is a clone. These two behaviours does not match.

So that we cannot deploy a new staking contract for existing vault, this may not be expected.

# 8. difference between code and WhitePaper

The WhitePaper states that 

>The protocol will not take any percentage of the fees that accrue in Vaults. Instead each Wrapper will be able to charge a small fee of a few basis points. 

But the Vaults' code has fee design such as management fee and performance fee. 

This difference should be handled by update the whitepaper or changing the code.

# 9. Vault fees can be bypassed by just deposit into Adapters

User can simply deposit into adapters to bypass vault management fee and performance fee.

Consider only allow vaults to be able to deposit into Adapters.