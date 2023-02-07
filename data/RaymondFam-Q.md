## Parameterized instead of hard coded
Immutable instances should be parameterized at the constructor instead of getting them directly assigned in the state variable declaration.

Here are some of the instances entailed:
 
[File: VaultController.sol#L36-L40](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L36-L40)

```solidity
  bytes32 public immutable VAULT = "Vault";
  bytes32 public immutable ADAPTER = "Adapter";
  bytes32 public immutable STRATEGY = "Strategy";
  bytes32 public immutable STAKING = "Staking";
  bytes4 internal immutable DEPLOY_SIG = bytes4(keccak256("deploy(bytes32,bytes32,bytes)"));
```
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

- at least a minimalist NatSpec is provided on [EIP165.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/EIP165.sol)
- @return is included in the function NatSpec of [`deploy()`](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneFactory.sol#L34-L39) in CloneFactory.sol 

## Typo mistakes
[File: AdminProxy.sol#L11](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L11)

```diff
- * @notice  Ownes contracts in the vault ecosystem to allow for easy ownership changes.
+ * @notice  Owner contracts in the vault ecosystem to allow for easy ownership changes.
```
[File: CloneRegistry.sol#L14](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L14)

```diff
- * Is used by `VaultController` to check if a target is a registerd clone.
+ * Is used by `VaultController` to check if a target is a registered clone.
```
[File: YearnAdapter.sol#L100](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L100)

```diff
-    /// @notice The amount of assets that are free to be withdrawn from the yVault after locked profts.
+    /// @notice The amount of assets that are free to be withdrawn from the yVault after locked profits.
```
[File: VaultController.sol#L87](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L87)

```diff
-   * @dev This function is the one stop solution to create a new vault with all necessary admin functions or auxiliery contracts.
+   * @dev This function is the one stop solution to create a new vault with all necessary admin functions or auxiliary contracts.
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
## Sanity check to prevent adding duplicate clone
Although only the owner can call `addClone()` in CloneRegistry.sol, it does not prevent accidental addition of duplicate clones. 

Consider having the function refactored as follows to ensure all clones pushed into the arrays are unique:

[File: CloneRegistry.sol#L41-L51](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L41-L51)

```diff
+ error CloneAlreadyAdded();

  function addClone(
    bytes32 templateCategory,
    bytes32 templateId,
    address clone
  ) external onlyOwner {
+    if (cloneExists[clone]) revert CloneAlreadyAdded();

    cloneExists[clone] = true;
    clones[templateCategory][templateId].push(clone);
    allClones.push(clone);

    emit CloneAdded(clone);
  }
```
## Use `delete` to clear variables
`delete a` assigns the initial value for the type to `a`. i.e. for integers it is equivalent to `a = 0`, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. Similarly, it can also be used to set an address to zero address or a boolean to false. It has no effect on whole mappings though (as the keys of mappings may be arbitrary and are generally unknown). However, individual keys and what they map to can be deleted: If `a` is a mapping, then `delete a[x]` will delete the value stored at x.

The delete key better conveys the intention and is also more idiomatic.

For instance, the `a = false` instance below may be refactored as follows:

[File: TemplateRegistry.sol#L75](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L75)

```diff
-    template.endorsed = false;
+    delete template.endorsed;
```
## Solidity's Style Guide on contract layout
According to Solidity's Style Guide below:

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

In order to help readers identify which functions they can call, and find the constructor and fallback definitions more easily, functions should be grouped according to their visibility and ordered in the following manner:

constructor, receive function (if exists), fallback function (if exists), external, public, internal, private

And, within a grouping, place the view and pure functions last.

Additionally, inside each contract, library or interface, use the following order:

type declarations, state variables, events, modifiers, functions

Consider adhering to the above guidelines for all contract instances entailed.

## Lines too long
Lines in source code are typically limited to 80 characters, but it’s reasonable to stretch beyond this limit when need be as monitor screens theses days are comparatively larger. Considering the files will most likely reside in GitHub that will have a scroll bar automatically kick in when the length is over 164 characters, all code lines and comments should be split when/before hitting this length. Keep line width to max 120 characters for better readability where possible.

Here are some of the instances entailed:

[File: AdapterBase.sol#L6](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L6)
[File: MultiRewardStaking.sol#L7](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L7)

## Interfaces and libraries should be separately saved and imported
Some contracts have interface(s) or libraries showing up in its/their entirety at the top/bottom of the contract facilitating an ease of references on the same file page. This has, in certain instances, made the already large contract size to house an excessive code base. Additionally, it might create difficulty locating them when attempting to cross reference the specific interfaces embedded elsewhere but not saved into a particular .sol file.

Consider saving the interfaces and libraries entailed respectively, and having them imported like all other files.

Here are some of the instances entailed:

[File: IYearn.sol#L32-L34](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/IYearn.sol#L32-L34)

