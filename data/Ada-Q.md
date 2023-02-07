Title: The code style is not consistent.

There were a lot of array-length-checks in the code. However, the implementations were not consistent. In some functions, it used `_verifyEqualArrayLength()` to check, but sometimes not. For example:

In VaultContoller, most functions used `_verifyEqualArrayLength()` to check whether length were match.
```
  function toggleTemplateEndorsements(bytes32[] calldata templateCategories, bytes32[] calldata templateIds)
    external
    onlyOwner
  {
    uint8 len = uint8(templateCategories.length);
    _verifyEqualArrayLength(len, templateIds.length);

...
    }
  }

```
However, in other functions, the function was not used.

```
  function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {
    uint256 len = targets.length;
    if (len != newPermissions.length) revert Mismatch();

...
  }
```

Moreover, the type of array length is not consistent, either. Sometimes the type was uint8, sometimes it was uint256. For example.
```
.//vault/PermissionRegistry.sol:39:    uint256 len = targets.length;
.//vault/VaultController.sol:314:    uint8 len = uint8(vaults.length);
.//vault/VaultController.sol:336:    uint8 len = uint8(vaults.length);
.//vault/VaultController.sol:353:    uint8 len = uint8(vaults.length);
.//vault/VaultController.sol:373:    uint8 len = uint8(vaults.length);
.//vault/VaultController.sol:436:    uint8 len = uint8(vaults.length);
.//vault/VaultController.sol:488:    uint8 len = uint8(vaults.length);
.//vault/VaultController.sol:517:    uint8 len = uint8(vaults.length);
.//vault/VaultController.sol:563:    uint8 len = uint8(templateCategories.length);
.//vault/VaultController.sol:583:    uint8 len = uint8(templateCategories.length);
.//vault/VaultController.sol:606:    uint8 len = uint8(vaults.length);
.//vault/VaultController.sol:619:    uint8 len = uint8(vaults.length);
.//vault/VaultController.sol:632:    uint8 len = uint8(vaults.length);
.//vault/VaultController.sol:645:    uint8 len = uint8(vaults.length);
.//vault/VaultController.sol:765:    uint8 len = uint8(adapters.length);
.//vault/VaultController.sol:805:    uint8 len = uint8(adapters.length);
.//vault/strategy/RewardsClaimer.sol:23:    uint256 len = rewardTokens.length;
.//vault/strategy/Pool2SingleAssetCompounder.sol:23:    uint256 len = rewardTokens.length;
.//vault/strategy/Pool2SingleAssetCompounder.sol:37:    uint256 len = rewardTokens.length;
.//vault/strategy/Pool2SingleAssetCompounder.sol:54:    uint256 len = rewardTokens.length;
.//vault/strategy/StrategyBase.sol:13:    uint8 len = uint8(sigs.length);
```

We recommend to keep the code style consistent.
