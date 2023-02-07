
# 1 Low Severity
## 1.1 Inclusion as start time can lead to unexpected behavior.

MultiRewardEscrow [uses the transaction inclusion time](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L114) (block.timestamp) as the starting point of the escrow. This can lead to unintended behavior in times of high transaction fee volatility. In case of sudden gas price increase, an adversary could also hold on to the transaction and re-publish it again when gas prices drop.

**Suggestion**: Let the user set the starting time of the escrow. It may also be interesting to add a mechanism to prevent republishing of the transaction, such as a nonce or inclusion deadline:

```diff
function lock(
    IERC20 token,
    address account,
    uint256 amount,
    uint32 duration,
-   uint32 offset
+   uint32 offset,
+   uint256 startTime
  ) external {
    ...

-    uint32 start = block.timestamp.safeCastTo32() + offset;
+    uint32 start = startTime + offset;

    ...
  }
```

## 1.2 Mass reward claim can be DoSed
[MultiRewardEscrow.claimRewards](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L160) and [MultiRewardStaking.claimRewards](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L174) revert if there is a single item with 0 claimable amount. This can make building convenience tools that claim for users hard to build and waste gas if the call is included after one of the items is individually claimed.

**Suggestion**: `continue` to the next item instead of `revert`ing so other they can continue being claimed:
```diff
  function claimRewards(bytes32[] memory escrowIds) external {
    for (uint256 i = 0; i < escrowIds.length; i++) {
			...

      uint256 claimable = _getClaimableAmount(escrow);
-      if (claimable == 0) revert NotClaimable(escrowId);
+      if (claimable == 0) continue;
      ...
    }
  }
```
```diff
  function claimRewards(address user, IERC20[] memory _rewardTokens) external accrueRewards(msg.sender, user) {
    for (uint8 i; i < _rewardTokens.length; i++) {
      ...
-      if (rewardAmount == 0) revert ZeroRewards(_rewardTokens[i]);
+      if (rewardAmount == 0) continue;
      ...
    }
  }
```

## 1.3 `isClaimable` returns true even for unclaimable escrows
If an escrow is in the offset period, it is not claimable. `MultiRewardEscrow.isClaimable`, however, will still [return true](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L1400), which is unexpected and can break potential future integrations:

**Suggestion**: Return false if the escrow is still in the offset period and document the function.

```diff
  function isClaimable(bytes32 escrowId) external view returns (bool) {
+   if (block.timestamp < escrows[escrowId].start) return false;
    return escrows[escrowId].lastUpdateTime != 0 && escrows[escrowId].balance > 0;
  }
```
## 1.4 CloneRegistry allows duplicate entries
[CloneRegistry.addClone](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneRegistry.sol#L41) does not check if the item was already added before, pushing a duplicate into the `clones` mapping and `allClones` array as well as emitting a new `CloneAdded` event which can confuse offchain tooling.

**Suggestion**: Disallow adding a clone that already exists:

```diff
  ...
+  error CloneAlreadyExists();
  ...
  function addClone(
    bytes32 templateCategory,
    bytes32 templateId,
    address clone
  ) external onlyOwner {
+   if(cloneExists[clone]) revert CloneAlreadyExists();
    cloneExists[clone] = true;
    clones[templateCategory][templateId].push(clone);
    allClones.push(clone);

    emit CloneAdded(clone);
  }
```
## 1.5 Anyone can add a template
According to the documentation, [DeploymentController.addTemplate](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/DeploymentController.sol#L66) should only be callable by the owner (`VaultController` or `AdminProxy`) but it is callable by anyone because it is missing a `onlyOwner` modifier.

**Suggestion**: Add the missing `onlyOwner` modifier:
```diff
  /**
   * @notice Adds a new category for templates. Caller must be owner. (`VaultController` via `AdminProxy`)
   * @param templateCategory Category of the new template.
   * @param templateId Unique Id of the new template.
   * @param template New template (See ITemplateRegistry for more details)
   * @dev (See TemplateRegistry for more details)
   */
  function addTemplate(
    bytes32 templateCategory,
    bytes32 templateId,
    Template calldata template
-  ) external {
+  ) external onlyOwner {
    templateRegistry.addTemplate(templateCategory, templateId, template);
  }
```
## 1.6 Harvest cooldown can have illegal value
Documentation and code on `setHarvestCooldown` [disallow >= 1 day durations](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L498), but the adapter can be initialized with any value.

**Suggestion**: Disallow initializing the adapter with illegal values:

```diff
    function __AdapterBase_init(bytes memory popERC4626InitData)
        internal
        onlyInitializing
    {
        ...
+       if (_harvestCooldown >= 1 days) revert InvalidHarvestCooldown(_harvestCooldown);
        harvestCooldown = _harvestCooldown;

        if (_strategy != address(0)) _verifyAndSetupStrategy(_requiredSigs);

				...
    }
```

# 2 Code QA
2.1 [IMultiRewardEscrow.Fee.accrued](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/IMultiRewardEscrow.sol#L26) field is never used.
2.2 [EIP165](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/EIP165.sol#L6): Use `interface` or `abstract` instead of `contract`.
2.3 [MultiRewardStaking](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L31), [BeefyAdapter](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L24): Instead of declaring `_name` and `_symbol` again, use the fields already available from the parent `ERC20` contracts by calling `__ERC20_init`  inside `initialize`.
2.4 [WithRewards](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/WithRewards.sol#L12): Add the missing  `abstract` in the contract declaration.
2.5 [AdapterBase.harvest](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L449): Only emit `Harvested` from inside the if since it does not actually harvest if the condition passes. This will also save gas.
# 3 Documentation QA
3.1 [ITemplateRegistry.Template](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/ITemplateRegistry.sol#L13): Remove trailing dots from natspec.
3.2 [AdminProxy](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/AdminProxy.sol#L11), [VaultController](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L47): Typo in documentation. Change `Ownes` to `Owns`.
3.3 [Vault.changeAdapter](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L592): Neither `highWaterMark` nor `assetsCheckpoint` are updated here. Remove the misleading docs.
3.4 [VaultController.deployVault](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L79): Caller does not need to be the owner if `address(1)` is set to `false`. Clarify this in the documentation.
3.5 [VaultController.deployAdapter](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L186): As in 3.4, caller does not need to be the owner if `address(1)` is set to `false`. Clarify this in the documentation.

