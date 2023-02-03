### 1. ```cloneFactory``` could be made as immutable.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L23
```
    ICloneFactory public cloneFactory;
```
It's only assigned when deploying the contract.

### 2. ```cloneRegistry``` could be made as immutable.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L24
```
    ICloneRegistry public cloneRegistry;
```
It's only assigned when deploying the contract.

### 3. ```templateRegistry``` could be made as immutable.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L25
```
    ITemplateRegistry public templateRegistry;
```
It's only assigned when deploying the contract.

### 4. ```VaultInitialized``` event could emit ```asset_``` instead of ```asset```.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L97
```
    emit VaultInitialized(contractName, address(asset));
```
```asset``` is read from storage.

### 5. ```quitPeriod``` is read twice from storage.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L541-L542
```
    if (block.timestamp < proposedFeeTime + quitPeriod)
    revert NotPassedQuitPeriod(quitPeriod);
```
Add a local variable for ```quitPeriod```.

### 6. ```proposedFees``` is read twice from storage.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L544-L545
```
    emit ChangedFees(fees, proposedFees);
    fees = proposedFees;
```
Add a local variable for ```proposedFees```.

### 7. ```adapter``` is read twice from storage.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L604-L606
```
    asset.approve(address(adapter), 0);

    emit ChangedAdapter(adapter, proposedAdapter);
```
Add a local variable for ```adapter```.

### 8. ```QuitPeriodSet``` event could have ```_quitPeriod``` instead of ```quitPeriod```.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L635
```
    quitPeriod = _quitPeriod;

    emit QuitPeriodSet(quitPeriod);
```
Could use a local variable ```_quitPeriod```.

### 9. ```uint8``` is not always cheaper than ```uint256```.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L314
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L353
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L436
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L488
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L517
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L583
```
src/vault/VaultController.sol
  314    uint8 len = uint8(vaults.length);
  315
  316    _verifyEqualArrayLength(len, newAdapter.length);

src/vault/VaultController.sol
  353    uint8 len = uint8(vaults.length);
  354
  355    _verifyEqualArrayLength(len, fees.length);

src/vault/VaultController.sol
  436    uint8 len = uint8(vaults.length);
  437    for (uint256 i = 0; i < len; i++) {

src/vault/VaultController.sol
  488    uint8 len = uint8(vaults.length);
  489
  490    _verifyEqualArrayLength(len, rewardTokens.length);
  491    _verifyEqualArrayLength(len, rewardsSpeeds.length);

src/vault/VaultController.sol
  517    uint8 len = uint8(vaults.length);
  518
  519    _verifyEqualArrayLength(len, rewardTokens.length);
  520    _verifyEqualArrayLength(len, amounts.length);

src/vault/VaultController.sol
  583    uint8 len = uint8(templateCategories.length);
  584    _verifyEqualArrayLength(len, templateIds.length);
```
```len``` could have ```uint256``` since it is converted to ```uint256``` in ```_verifyEqualArrayLength()``` function.

### 10. Some variables could be made as immutable.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L387
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L535
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L717
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L822
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L191
```
src/vault/VaultController.sol
  387    IVaultRegistry public vaultRegistry;

src/vault/VaultController.sol
  535    IMultiRewardEscrow public escrow;

src/vault/VaultController.sol
  717    IAdminProxy public adminProxy;

src/vault/VaultController.sol
  822    IPermissionRegistry public permissionRegistry;

src/utils/MultiRewardEscrow.sol
  191    address public feeRecipient;
```
They are only assigned when deploying the contract.

### 11. Some attributes could be removed from the struct.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVaultRegistry.sol#L16-L21
```
    /// @notice OPTIONAL - If the asset is an Lp Token these are its underlying assets
    address[8] swapTokenAddresses;
    /// @notice OPTIONAL - If the asset is an Lp Token its the pool address
    address swapAddress;
    /// @notice OPTIONAL - If the asset is an Lp Token this is the identifier of the exchange (1 = curve)
    uint256 exchange;
```
```swapTokenAddresses```, ```swapAddress```, and ```exchange``` are never used from the contracts.

### 12. ```nominatedOwner``` is read multiple times from storage.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/Owned.sol#L23-L25
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/OwnedUpgradeable.sol#L25-L27
```
src/utils/Owned.sol
  23    require(msg.sender == nominatedOwner, "You must be nominated before you can accept ownership");
  24    emit OwnerChanged(owner, nominatedOwner);
  25    owner = nominatedOwner;

src/utils/OwnedUpgradeable.sol
  25    require(msg.sender == nominatedOwner, "You must be nominated before you can accept ownership");
  26    emit OwnerChanged(owner, nominatedOwner);
  27    owner = nominatedOwner;
```

Add a local variable for ```nominatedOwner```.