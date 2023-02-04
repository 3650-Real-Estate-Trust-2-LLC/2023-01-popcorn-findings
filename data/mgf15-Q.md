there is some mess for a onlyOwner modifire ! 

DeploymentController.sol#66 addTemplate function 

```
function addTemplate(
    bytes32 templateCategory,
    bytes32 templateId,
    Template calldata template
  ) external {
    templateRegistry.addTemplate(templateCategory, templateId, template);
  }
```

VaultController.sol#605 

pauseAdapters & pauseVaults mess onlyOwner modifire ! any one can pause it !
```
  function pauseAdapters(address[] calldata vaults) external {
    uint8 len = uint8(vaults.length);
    for (uint256 i = 0; i < len; i++) {
      _verifyCreatorOrOwner(vaults[i]);
      (bool success, bytes memory returnData) = adminProxy.execute(
        IVault(vaults[i]).adapter(),
        abi.encodeWithSelector(IPausable.pause.selector)
      );
      if (!success) revert UnderlyingError(returnData);
    }
  } 
 function pauseVaults(address[] calldata vaults) external {
    uint8 len = uint8(vaults.length);
    for (uint256 i = 0; i < len; i++) {
      _verifyCreatorOrOwner(vaults[i]);
      (bool success, bytes memory returnData) = adminProxy.execute(
        vaults[i],
        abi.encodeWithSelector(IPausable.pause.selector)
      );
      if (!success) revert UnderlyingError(returnData);
    }
  }
```

also unpauseAdapters & unpauseVaults are mess onlyOwner modifire attacker can unpauseAdapters and unpauseVaults ! 
on the same file VaultController.sol#631
```
  function unpauseAdapters(address[] calldata vaults) external {
    uint8 len = uint8(vaults.length);
    for (uint256 i = 0; i < len; i++) {
      _verifyCreatorOrOwner(vaults[i]);
      (bool success, bytes memory returnData) = adminProxy.execute(
        IVault(vaults[i]).adapter(),
        abi.encodeWithSelector(IPausable.unpause.selector)
      );
      if (!success) revert UnderlyingError(returnData);
    }
  }

  /// @notice Unpause deposits. Caller must be owner or creator of the Vault.
  function unpauseVaults(address[] calldata vaults) external {
    uint8 len = uint8(vaults.length);
    for (uint256 i = 0; i < len; i++) {
      _verifyCreatorOrOwner(vaults[i]);
      (bool success, bytes memory returnData) = adminProxy.execute(
        vaults[i],
        abi.encodeWithSelector(IPausable.unpause.selector)
      );
      if (!success) revert UnderlyingError(returnData);
    }
  }
```

Vault.sol#594 

changeAdapter function mess onlyowner modifire 
```
function changeAdapter() external takeFees {
        if (block.timestamp < proposedAdapterTime + quitPeriod)
            revert NotPassedQuitPeriod(quitPeriod);

        adapter.redeem(
            adapter.balanceOf(address(this)),
            address(this),
            address(this)
        );
```