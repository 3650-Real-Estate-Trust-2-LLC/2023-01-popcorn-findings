# The `owner` arguments for `Vault.initialize()`, `withdraw()`, `redeem()` and `permit()` shadows the `owner()` function on `OwnedUpgradeable`

The `owner` argument should follow the same style as the other arguments to prevent shadowing the parent function. Using a different variable name/argument will prevent shadowing.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L57

Examples:
```js
    function initialize(
        IERC20 asset_,
        IERC4626 adapter_,
        VaultFees calldata fees_,
        address feeRecipient_,
        address owner_ // change here
    ) external initializer {
        // ...
        __Owned_init(owner_); // and here
```

```js
    function withdraw(
        uint256 assets,
        address receiver,
        address assetOwner
    )
```
