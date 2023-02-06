1. Use openzeppelin's ECDSA library for signature verification rather than reinventing the wheel.
2. Utilize openzeppelin's EIP712.sol for generating structured hashed data and preventing replay attacks for signature verification.
3. Restrict function state mutability to view on `https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L242` and `https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L673`.
4. Use curly braces around if blocks to improve readability.
