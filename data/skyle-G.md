G1 - remove unused variables
src/vault/Vault.sol:Vault line 448
uint256 highWaterMark_ = highWaterMark; 

G2 - reduce stack variables
// this currentAssets only used once, we can just call the function totalAssets()
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L483
uint256 currentAssets = totalAssets();


