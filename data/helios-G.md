# 1. `public` functions not called by the contract should be declared `external` instead

Link: https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/EIP165.sol#L6

Contracts are allowed to override their parents' functions and change the visibility from `external` to `public`.

