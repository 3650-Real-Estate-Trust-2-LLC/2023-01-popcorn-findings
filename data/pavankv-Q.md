## 1 . Include return parameters in NatSpec comments

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

If Return parameters are declared, you must prefix them with ”/// @return”.

Some code analysis programs do analysis by reading NatSpec details, if they can’t see the “@return” tag, they do incomplete analysis.
Recommended Mitigation Steps
Include return parameters in NatSpec comments

codesnippet:-
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L89
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L110
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L129
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L147
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L173
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L193
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L203
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L242


## 2 .Add Zero check for shares :-

code snippet:-
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L129
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L118
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L129


## 3 . modifiers should be use only for external or public not internal functions :-
modifiers should be use only for external or public not internal functions because internal function can call by internal only so add to deposit() in line AdapterBase.sol#L110. Internals cannot called by external users so add to exteranl function only to mint() , deposit() and other . 


code snippet:-
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L152


## 4 .Events should emit after operation :-

In below mentioned code emit should after operation because if adapter didn't approve but event emitted changed adapter .

code snippeT:-
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L606


## 5.Remove empty blocks to save gas:-
Remove empty blocks because while deploying even single letter will consume the gas .

code snippet:-
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L477
