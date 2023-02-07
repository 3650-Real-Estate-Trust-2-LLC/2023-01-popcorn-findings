## [QA Report]

### [NC-1] Unused imports 
There are 2 instances of it
its recommend to remove unused import for better redability of the code
[YearnAdapter.sol#L6](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L6) here `IStrategy` & `IAdapter` are unused

### [NC-2] Open TODO
There is 1 instance of it  and the permalink is
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L516
its recommend to complete it  before moving to deployment

### [NC-3] Missing NatSpec
There is a missing NatSpec comments on the `_handleInitialDeposit` function
here is the permalink of it
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L164

### [NC-4] Use of a more recent solidity version 
The current version of all contracts are set to ^0.8.15;
its recommend to use more stable version & according to [SWC-103](https://swcregistry.io/docs/SWC-103) its say that
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.


