*approve max has risk:
Consider changing asset.approve(address(adapter_), type(uint256).max)  to approve only the required amount
 rather use `increaseERC20Allowance()` and `decreaseERC20Allowance()`.
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#80

*fee must be different than zero
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#88
