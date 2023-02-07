This is only a partial report - it is not an exhaustive list of issues with the codebase.

## Inconsistent number of indentation spaces

Some files, such as VaultController.sol use 2 spaces, whereas other files such as Vault.sol use 4.

## Very poor code commenting

In general there are very few comments throughout the codebase to explain how it works. When writing smart contracts it is essential that every single detail is explained explicitly.

## Inconsistent naming

Compared to the whitepaper, the codebase uses incorrect terms to refer to base concepts. This makes it hard to understand the codebase.

The whitepaper refers to `Wrapper`, but the codebase uses the term `Adapter`.

The whitepaper refers to `Modules`, but the codebase uses the term `Strategy`.

## The purpose of immutables in VaultController is not defined

```
  bytes32 public immutable VAULT = "Vault";
  bytes32 public immutable ADAPTER = "Adapter";
  bytes32 public immutable STRATEGY = "Strategy";
  bytes32 public immutable STAKING = "Staking";
```

It needs to be explained in a comment that these are contract template categories.

## Contract methods do not comment which interface they are implementing, or contract they are overriding.

For example, YearnAdapter.maxDeposit() is implementing IERC4626Upgradeable.maxDeposit()

This needs to be put in the comment above the function.

## `VaultController.deploymentController` being passed redundantly

`VaultController.deployVault()` passes `VaultController.deploymentController` to `VaultController._deployVault()`

This is not necessary because `VaultController.deploymentController` is available to every method in the contract.

`VaultController.deployAdapter` and `VaultController.deployStaking` do the same thing.

If it is implemented this way to save gas, this need to be documented in a code comment.

## Math for calculating fees is not explained

In `Vault.deposit()`, `feeShares` is calculated from `assets` and `depositFee` as follows:

```
uint256 feeShares = convertToShares(
    assets.mulDiv(uint256(fees.deposit), 1e18, Math.Rounding.Down)
);
```

This is not explained, but it implies that a fee of 1e18 is equivalent to 100%. 

In `Vault.mint()`, `feeShares` is calculated from `shares` and `depositFee` as follows:

```
uint256 depositFee = uint256(fees.deposit);
uint256 feeShares = shares.mulDiv(
    depositFee,
    1e18 - depositFee,
    Math.Rounding.Down
);
```

This is the inverse of the math in deposit(), but it needs to be explained why this is correct.

Similar issues exists in calculations involving `withdrawalFee`, `managementFee` and `performanceFee`.


## Duplicated logic in `Vault.deposit()` and `Vault.mint()`

These two functions end up doing the same thing.

Logic at the end of these functions could be replaced by the following internal function:

```
function _deposit(uint256 assets, uint256 shares, uint256 feeShares, address receiver)
    internal
{
  if (feeShares > 0) _mint(feeRecipient, feeShares);
  // Mint vault tokens for the receiver.
  _mint(receiver, shares);
  // Attempt to transfer the underlying token from sender to the vault.
  asset.safeTransferFrom(msg.sender, address(this), assets);
  // Send the assets from the vault to the adapter (also an ERC4626).
  adapter.deposit(assets, address(this));
  // Log event.
  emit Deposit(msg.sender, receiver, assets, shares);
}
```

## `PermissionRegistry` utilizes a strange hack

`PermissionRegistry` enables addresses to be endorsed, rejected.

`VaultController._verifyToken()` can operate in one of two modes.

  * whitelist - If `address(0)` has been endorsed, only token addresses that have been endorsed are acceptable.
  * blacklist - If `address(0)` has not been endorsed, token addresses must not have been rejected 

`VaultController.canCreate()` operates in a similar manner, except it checks `address(1)` to determine the operating mode.

This looks like it might have been code not intended for production. If it is intended that the permission operating mode can be switched over, this needs to be implemented as a proper API in PermissionRegistry instead of this strange hack.

Changing from blacklisting to whitelisting could totally screw up the address permissions. This should probably be set at deploy time only.

## `CloneFactory.deploy()` is badly written

It checks the `success` variable even if `template.requiresInitData` is false.

It could be written more robustly like this:

```
		if (template.requiresInitData) {
			bool success;
			(success, ) = clone.call(data);
	    if (!success) revert DeploymentInitFailed();
		}
```

## `Vault` does not specify that it implements IERC4626

`Vault` correctly specifies that it is `ERC20Upgradeable`, which is `IERC20Upgradeable`.
`Vault` does not specify that it implements `IERC4626`, even though that is its core purpose.

