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

# Vault contract implementation does not disable initializers

The Vault.sol contract should implement `_disableInitializers()` in its constructure to prevent implementation contracts from being initialized. As this contract does not use `selfdestruct()` the risk is minimized but could still lead to user confusion if it is taken over by a third party.

https://docs.openzeppelin.com/contracts/4.x/api/proxy#Initializable-_disableInitializers--

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L26

Proposed:
```js
constructor() {
  _disableInitializers();
}
```
