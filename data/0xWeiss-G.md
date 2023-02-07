# [G-01] EncodePacked is cheaper than string.concat

## Summary
The cost of the abi.encodePacked method is generally lower than the one of string.concat because it performs a more efficient encoding of the input data. 

## Code Snippet
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol?plain=1#L70
## Tool used

Manual Review

## Recommendation
Use  abi.encodePacked instead of string.concat