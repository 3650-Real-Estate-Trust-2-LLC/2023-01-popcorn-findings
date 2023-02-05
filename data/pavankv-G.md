## 1 . Use named returns for local variables where it is possible :-

code snippet:-
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L114
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L133
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L177
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L197

## 2 .Using Solidity version 0.8.17 will provide an overall gas optimization

Using at least 0.8.10 will save gas due to skipped extcodesize check if there is a return value. Currently the contracts are compiled using version 0.8.7 (Foundry default). It is easily changeable to 0.8.17 using the command sed -i 's/0\.8\.7/^0.8.0/' test/*.sol && sed -i '4isolc = "0.8.17"' foundry.toml. This will have the following total savings obtained by forge snapshot --diff | tail -1:

code snippet:-
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L3
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L4



## 3 .Increments/decrements can be unchecked in for-loops

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline

code snippet:-
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L319
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L337
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L357
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L374
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L494
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L564
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L587
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L607