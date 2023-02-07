# SECONDS_PER_YEAR don't conform to **accruedManagementFee's** comment
[uint256 constant SECONDS_PER_YEAR = 365.25 days;](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L35)
while: [365\*24\*60=525600](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L425)

# functions in VaultController.sol cast input array's length which might revert if input array's length larger than 256
there should be 15 places


    ./vault/VaultController.sol-  function proposeVaultAdapters(address[] calldata vaults, IERC4626[] calldata newAdapter) external {
    ./vault/VaultController.sol:    uint8 len = uint8(vaults.length);
    --
    ./vault/VaultController.sol-  function changeVaultAdapters(address[] calldata vaults) external {
    ./vault/VaultController.sol:    uint8 len = uint8(vaults.length);
    --
    ./vault/VaultController.sol-  function proposeVaultFees(address[] calldata vaults, VaultFees[] calldata fees) external {
    ./vault/VaultController.sol:    uint8 len = uint8(vaults.length);
    --
    ./vault/VaultController.sol-  function changeVaultFees(address[] calldata vaults) external {
    ./vault/VaultController.sol:    uint8 len = uint8(vaults.length);
