G1 - remove unused variables
src/vault/Vault.sol:Vault line 448
uint256 highWaterMark_ = highWaterMark; 

G2 - reduce stack variables
// this currentAssets only used once, we can just call the function totalAssets()
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L483
uint256 currentAssets = totalAssets();

G3 use uint256 instead of uint8 
https://ethereum.stackexchange.com/questions/3067/why-does-uint8-cost-more-gas-than-uint256

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L336
uint8 len = uint8(vaults.length);
  for (uint8 i = 0; i < len; i++) {
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L373
uint8 len = uint8(vaults.length);
  for (uint8 i = 0; i < len; i++) {
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L436
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L448
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L517
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L563
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L583
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L606
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L619
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L632
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L645
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L765



