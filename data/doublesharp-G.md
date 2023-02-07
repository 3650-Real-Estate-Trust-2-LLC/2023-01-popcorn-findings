# Save gas by using an if statement when deploying contract in CloneFactory

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L106

Current:
```
  function deploy(Template calldata template, bytes calldata data) external onlyOwner returns (address clone) {
    clone = Clones.clone(template.implementation);

    bool success = true;
    if (template.requiresInitData) (success, ) = clone.call(data);

    if (!success) revert DeploymentInitFailed();

    emit Deployment(clone);
  }
```

Proposed:
```
  function deploy(Template calldata template, bytes calldata data) external onlyOwner returns (address clone) {
    clone = Clones.clone(template.implementation);

    if (template.requiresInitData) {
      (bool success, ) = clone.call(data);
      if (!success) revert DeploymentInitFailed();
    }

    emit Deployment(clone);
  }
```

# Use a decrementing loop to take advantage of ISZERO check

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/strategy/RewardsClaimer.sol#L25

Current:
```
uint256 len = rewardTokens.length;
// send all tokens to destination
for (uint256 i = 0; i < len; i++) {
  ERC20 token = ERC20(rewardTokens[i]);
  uint256 amount = token.balanceOf(address(this));

  token.safeTransfer(rewardDestination, amount);

  emit ClaimRewards(address(token), amount);
}
```

Proposed:
```
// send all tokens to destination
for (uint256 i = rewardTokens.length; i != 0; ) {
  unchecked{ --i; }
  ERC20 token = ERC20(rewardTokens[i]);
  uint256 amount = token.balanceOf(address(this));

  token.safeTransfer(rewardDestination, amount);

  emit ClaimRewards(address(token), amount);
}
```

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/strategy/StrategyBase.sol#L13

Current:
```
uint8 len = uint8(sigs.length);
for (uint8 i; i < len; i++) {
  if (sigs[i].length == 0) return;
  if (!IEIP165(address(this)).supportsInterface(sigs[i])) revert FunctionNotImplemented(sigs[i]);
}
```

Iterator should be a uint256 as downcasting wastes gas.

Proposed:
```
for (uint256 i = sigs.length; i != 0; ) {
  unchecked{ --i; }
  if (sigs[i].length == 0) return;
  if (!IEIP165(address(this)).supportsInterface(sigs[i])) revert FunctionNotImplemented(sigs[i]);
}
```

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/strategy/Pool2SingleAssetCompounder.sol#L23

Current:
```
uint256 len = rewardTokens.length;
// send all tokens to destination
for (uint256 i = 0; i < len; i++) {
  uint256 amount = ERC20(rewardTokens[i]).balanceOf(address(this));

  if (amount > 0) {
    tradePath[0] = rewardTokens[i];

    IUniswapRouterV2(router).swapExactTokensForTokens(amount, 0, tradePath, address(this), block.timestamp);
  }
}
```

Proposed:
```
uint256 len = rewardTokens.length;
// send all tokens to destination
for (uint256 i = rewardTokens.length; i !=0; ) {
  unchecked{ --i; }

  uint256 amount = ERC20(rewardTokens[i]).balanceOf(address(this));

  if (amount > 0) {
    tradePath[0] = rewardTokens[i];
    IUniswapRouterV2(router).swapExactTokensForTokens(amount, 0, tradePath, address(this), block.timestamp);
  }
}
```
