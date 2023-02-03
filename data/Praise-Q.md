#### IN VAULTCONTROLLER, https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L666-L676
The following functions were supposed to check if the caller was the creator of the vault or owner. but the check happens only after the calls, so it is ineffective.
```
  /// @notice Verify that the caller is the creator of the vault or owner of `VaultController` (admin rights).
  function _verifyCreatorOrOwner(address vault) internal returns (VaultMetadata memory metadata) {
    metadata = vaultRegistry.getVault(vault);
    if (msg.sender != metadata.creator || msg.sender != owner) revert NotSubmitterNorOwner(msg.sender);
  }


  /// @notice Verify that the caller is the creator of the vault.
  function _verifyCreator(address vault) internal view returns (VaultMetadata memory metadata) {
    metadata = vaultRegistry.getVault(vault);
    if (msg.sender != metadata.creator) revert NotSubmitter(msg.sender);
  }
```