1. Unused local variable.
```
   --> src/vault/Vault.sol:448:9:
    |
448 |         uint256 highWaterMark_ = highWaterMark;
    |         ^^^^^^^^^^^^^^^^^^^^^^

```

2. Unused function parameter.

The `vault` parameter is not used.
```
   --> src/vault/VaultController.sol:390:27:
    |
390 |   function _registerVault(address vault, VaultMetadata memory metadata) internal {
    | 
```

3. Functions can be restricted to view.
```
   --> src/utils/MultiRewardStaking.sol:351:3:
    |
351 |   function _calcRewardsEnd(

   --> src/vault/VaultController.sol:242:3:
    |
242 |   function _encodeAdapterData(DeploymentArgs memory adapterData, bytes memory baseAdapterData)
   
   --> src/vault/VaultController.sol:667:3:
    |
667 |   function _verifyCreatorOrOwner(address vault) internal returns (VaultMetadata memory metadata) {
   
   --> src/vault/strategy/StrategyBase.sol:12:3:
   |
12 |   function verifyAdapterSelectorCompatibility(bytes4[8] memory sigs) public {
   
   --> src/vault/strategy/Pool2SingleAssetCompounder.sol:14:3:
   |
14 |   function verifyAdapterCompatibility(bytes memory data) public override {

```

4. Don't initialize variables with default value.
```
function deploy(Template calldata template, bytes calldata data) external onlyOwner returns (address clone) {
    clone = Clones.clone(template.implementation);

    bool success = true;
    if (template.requiresInitData) (success, ) = clone.call(data);

    if (!success) revert DeploymentInitFailed();

    emit Deployment(clone);
  }


===>
   if (template.requiresInitData) (bool success, ) = clone.call(data);
```

