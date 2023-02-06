## Unspecific compiler version pragma
For some source-units the compiler version pragma is very unspecific, i.e. ^0.8.15. While this often makes sense for libraries to allow them to be included with multiple different versions of an application, it may be a security risk for the actual application implementation itself. A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up actually checking a different EVM compilation that is ultimately deployed on the blockchain.

Avoid floating pragmas where possible. It is highly recommend pinning a concrete compiler version (latest without security issues) in at least the top-level “deployed” contracts to make it unambiguous which compiler version is being used. Rule of thumb: a flattened source-unit should have at least one non-floating concrete solidity compiler version pragma.

## Use a more recent version of solidity
The protocol adopts version 0.8.15 on writing contracts. For better security, it is best practice to use the latest Solidity version, 0.8.17.

Security fix list in the versions can be found in the link below:

https://github.com/ethereum/solidity/blob/develop/Changelog.md

## Inadequate NatSpec
Solidity contracts can use a special form of comments, i.e., the Ethereum Natural Language Specification Format (NatSpec) to provide rich documentation for functions, return variables and more. Please visit the following link for further details:

https://docs.soliditylang.org/en/v0.8.16/natspec-format.html

For instance, it will be of added values to the users and developers if:

- a comprehensive NatSpec is provided on [EIP165.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/EIP165.sol)

## Typo mistakes
[File: AdminProxy.sol#L11](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L11)

```diff
- * @notice  Ownes contracts in the vault ecosystem to allow for easy ownership changes.
+ * @notice  Owner contracts in the vault ecosystem to allow for easy ownership changes.
```
## Return statement on named returns

Functions with named returns should not have a return statement to avoid unnecessary confusion.

For instance, the following function may be refactored as:

[File: AdminProxy.sol#L19-L25](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L19-L25)

```diff
  function execute(address target, bytes calldata callData)
    external
    onlyOwner
    returns (bool success, bytes memory returndata)
  {
-    return target.call(callData);
+    (success, returndata) = target.call(callData);
  }
```