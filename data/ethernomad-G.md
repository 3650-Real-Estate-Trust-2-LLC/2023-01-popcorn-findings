## Over-use of `MathUpgradeable.mulDiv()`

`mulDiv()` is very expensive. While this function can be essential to avoid overflow, there are situations where its use can be avoid entirely.

For example, fees are stored as a uint256 where 1e18 is 100%. This is overkill. They could be stored as an integer that the amount can be divided by. A regular EVM division could be used for this.

## `Vault.deposit()` wastes gas.

`deposit(uint256 assets) public` calls `deposit(uint256 assets, address receiver) public`

Calling a public function instead of internal wastes gas.

`deposit(uint256 assets, address receiver)` then goes on to check `receiver == address(0)` unnecessarily wasting more gas.

A better arrangement would for both `deposit(uint256 assets)` and `deposit(uint256 assets, address receiver)` to call an internal function.

`receiver == address(0)` would only be checked in `deposit(uint256 assets, address receiver)`.

`mint()`, `withdraw()` and `redeem()` have the same issue.