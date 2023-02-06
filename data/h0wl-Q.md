# Incorrect NatSpec comment

The code and comments in `DeploymentController` are not consistent. While the comment states it should only be called by the owner, the function `addTemplate` is callable by anyone:
Comment: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L60
Code: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L70

So at first, it looks like a permission error where `onlyOwner` check is missing however the reason I believe its only QA finding is due the project description that templates can be added by anyone and e.g. the comment in here:
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L15

And in `TemplateRegistry` on line https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L71 the `onlyOwner` check is performed.

Similar cases:
`VaultController` function `deployVault`:
Comment: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L79
Code: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L97

`VaultController` function `deployAdapter`:
Comment: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L181
Code: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L191

`VaultController` function `deployStaking`:
Comment: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L274
Code: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L278

Recommendation: Make the comments and code consistent, or add the `onlyOwner` check if it should be there.

## Floating pragma version

Contracts should be deployed with a locked and up to date compiler version that they have been tested with thoroughly. It will help to ensure that the contracts are not deployed with an outdated Solidity compiler version that is known to have vulnerabilities or can cause unexpected behaviour.

In all contracts: `pragma solidity ^0.8.15;`

Recommendation: Lock the pragma version

Reference: https://swcregistry.io/docs/SWC-103 