# 1: ABI.ENCODE() IS LESS EFFICIENT THAN ABI.ENCODEPACKED()

Optimization details

### Context:

Use abi.encodePacked() where possible to save gas

## Proof of Concept

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L219 

## Tools Used

Manual Analysis
