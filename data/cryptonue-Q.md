## [L] `CloneRegistry` might contain duplicates of cloned address

```solidity
File: CloneRegistry.sol
41:   function addClone(
42:     bytes32 templateCategory,
43:     bytes32 templateId,
44:     address clone
45:   ) external onlyOwner {
46:     cloneExists[clone] = true;
47:     clones[templateCategory][templateId].push(clone);
48:     allClones.push(clone);
49:
50:     emit CloneAdded(clone);
51:   }
```

the `addClone` function didn't check if the `clone` address already in the `allClones` array. The function will just add the `clone` to the `allClones` array (Line:48). This will effect in duplicate exist when getting the `getAllClones()`.

It's better to check if the clone exist before adding it.

For example, adding the check in the `addClone()`

```solidity
if(cloneExists[clone]) revert CloneExists();
```

This simmilar check already implemented on the `TemplateRegistry.sol` in `addTemplateCategory` and `addTemplate` function.

## [L] `addRewardToken` exist but `removeRewardToken` doesn't exist

This missing of remove an entry of `rewardTokens` will make some potential issues, for example:

If owner wrongly input a token address, then when `claimRewards()` the `_rewardTokens` parameter should be filter out this wrongly token address, if not it will revert the `claimReward()` because of this wrong token.

We can make the `removeRewardToken()` follow some restriction to make sure there should be some reward / locked reward, (not blindly remove any existing token)

## [L] Doesn't check the amount

```solidity
File: RewardsClaimer.sol
25:     for (uint256 i = 0; i < len; i++) {
26:       ERC20 token = ERC20(rewardTokens[i]);
27:       uint256 amount = token.balanceOf(address(this));
28:
29:       token.safeTransfer(rewardDestination, amount);
30:
31:       emit ClaimRewards(address(token), amount);
32:     }
```

The `harvest()` function doesn't check if `amount` > 0, resulting it will transfer 0 amount of token, which it should be better to skip the transfer.

## [N] Repeated pattern of looping through address and calling `adminProxy.execute()` can be generalized/abstracted

Compacting a pause/unpause Adapters and Vaults. The 4 function of Pausing logic inside `VaultController.sol` can be compacted into a simple and gas efficient function.

- pauseAdapters
- pauseVaults
- unpauseAdapters
- unpauseVaults

All of 4 functions basically doing the same with a slight change in `adminProxy.execute()` parameters (address, bytes).

Moreover the repeated pattern of looping through address and calling `adminProxy.execute()` also exist in

- setAdapterPerformanceFees
- setAdapterHarvestCooldowns
- changeVaultAdapters
- changeVaultFees

we might abstract this kind of functions

## [N] Wrong Comments information

```solidity
File: VaultController.sol
792:     // Dont wait more than X seconds
793:     if (newCooldown > 1 days) revert InvalidHarvestCooldown(newCooldown);
```

The comment should be `Dont wait more than 1 day`

## [N] UNSPECIFIC COMPILER VERSION PRAGMA

```solidity
pragma solidity ^0.8.15;
```

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

## [N] SIGNATURE MALLEABILITY OF EVM’S ECRECOVER()

```solidity
address recoveredAddress = ecrecover(
    keccak256(
        abi.encodePacked(
            "\x19\x01",
            DOMAIN_SEPARATOR(),
            keccak256(
                abi.encode(
                    keccak256(
                        "Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"
                    ),
                    owner,
                    spender,
                    value,
                    nonces[owner]++,
                    deadline
                )
            )
        )
    ),
    v,
    r,
    s
);
```

`permit()` function in Vault.sol, AdapterBase.sol and MultiRewardStaking.sol is using `ecrecover()` function directly to verify the given signatures. However, the `ecrecover()` EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

Although a replay attack seems not possible for this project, but still, it's recommend to use the battle-tested OpenZeppelin’s ECDSA library. (Ensure using a version > 4.7.3 for there was a critical bug >= 4.1.0 < 4.7.3)
