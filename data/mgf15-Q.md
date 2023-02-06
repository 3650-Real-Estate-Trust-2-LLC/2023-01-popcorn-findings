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
VaultController.sol#L335 is messing _verifyCreator();

```
  function changeVaultAdapters(address[] calldata vaults) external {
    uint8 len = uint8(vaults.length);
    for (uint8 i = 0; i < len; i++) {
      (bool success, bytes memory returnData) = adminProxy.execute(
        vaults[i],
        abi.encodeWithSelector(IVault.changeAdapter.selector)
      );
      if (!success) revert UnderlyingError(returnData);
    }
  }
```
line 372
```
function changeVaultFees(address[] calldata vaults) external {
    uint8 len = uint8(vaults.length);
    for (uint8 i = 0; i < len; i++) {
      (bool success, bytes memory returnData) = adminProxy.execute(
        vaults[i],
        abi.encodeWithSelector(IVault.changeFees.selector)
      );
      if (!success) revert UnderlyingError(returnData);
    }
  }
```