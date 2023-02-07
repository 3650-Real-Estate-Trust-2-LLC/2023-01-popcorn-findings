### File - Line No
[Vault.sol - L187](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L187)

## Miscalculation in mint() can lead to insolvent positions

The mint function uses convertToAssets() which rounds down the given number of shares to calculate a lesser number of assets. 

According to the documentation of ERC4626 and as performed in the previewMint(), the mint function must also round up the conversion of shares to assets such that the vault is never forced to mint more shares than the asset it holds.


### Recommendation:

The mint() must ensure that either be the first user of the Vault or in any other case, the minted shares are always less than equal to the number of assets held by the Vault. Therefore, rounding up the shares during the conversion to asset must ensure that the shares are always constrained.