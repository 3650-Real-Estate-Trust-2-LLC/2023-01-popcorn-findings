Error message too long in Owned.sol contract
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/Owned.sol#L23
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/Owned.sol#L35

and OwnedUpgradeable.sol   contract

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/OwnedUpgradeable.sol#L25
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/OwnedUpgradeable.sol#L37

shortening the revert strings to fit in 32 bytes, or using custom errors as described next.
Custom errors from Solidity 0.8.4 are cheaper than revert strings (cheaper deployment cost and runtime cost when the revert condition is met) Change all require to custom errors.

--------------------------------------------------------------------------------------------------------------------------------
Change Inequality sign In Vault.sol Contract

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L273

 >= IS CHEAPER THAN > (AND <= CHEAPER THAN <)
 Strict inequalities (>) are more expensive than non-strict ones (>=). This is due to some supplementary checks (ISZERO, 3 gas). This also holds true between <= and <.


--------------------------------------------------------------------------------------------------------------------------------------------
Variable declared but not used in vault.sol contract

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L448

The local variable highWaterMark_ should be used for operation. instead of reading storage multiple times in the operation here with highWaterMark.

--------------------------------------------------------------------------------------------
Consider adding a non-zero-value check in some function in the vault contract:

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L134
(assets) value should be checked.

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L170
(shares) value should be checked

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L211
(shares) value should be checked

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L253
(shares) value should be checked

Checking non-zero values can avoid an expensive external call and save gas.

-------------------------------------------------------------------------------------------------



