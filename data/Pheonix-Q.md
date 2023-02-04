## **[L-01] Use safeApprove**

- Pool2SingleAssetCompounder.sol:39

```markdown
ERC20(rewardTokens[i]).approve(router, type(uint256).max);
```

[https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/strategy/Pool2SingleAssetCompounder.sol#L39](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/strategy/Pool2SingleAssetCompounder.sol#L39)

- Vault.sol:80

```markdown
asset.approve(address(adapter_), type(uint256).max);
```

[https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L80](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L80)

- Vault.sol:604

```markdown
asset.approve(address(adapter), 0);
```

[https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L604](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L604)

- Vault.sol:610

```markdown
asset.approve(address(adapter), type(uint256).max);
```

[https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L610](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L610)

- VaultController.sol:171

```markdown
asset.approve(address(target), initialDeposit);
```

[https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L171](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L171)

- VaultController.sol:456

```markdown
IERC20(rewardsToken).approve(staking, type(uint256).max);
```

[https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L456](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L456)

## **[L-02] Use safeTransfer**

- VaultController.sol:457

```markdown
IERC20(rewardsToken).transferFrom(msg.sender, address(adminProxy), amount);
```

[https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L457](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L457)

- VaultController.sol:526

```markdown
rewardTokens[i].transferFrom(msg.sender, address(this), amounts[i]);
```

[https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L526](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L526)