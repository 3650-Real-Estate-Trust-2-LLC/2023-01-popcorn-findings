[L-01] Low level call result not correctly checked

https://github.com/code-423n4/2023-01-popcorn/blob/36477d96788791ff07a1ba40d0c726fb39bf05ec/src/vault/VaultController.sol#L238

The last adminProxy.execute is not checked for success.

```solidity
  function __deployAdapter(
    DeploymentArgs memory adapterData,
    bytes memory baseAdapterData,
    IDeploymentController _deploymentController
  ) internal returns (address adapter) {
    (bool success, bytes memory returnData) = adminProxy.execute(
      address(_deploymentController),
      abi.encodeWithSelector(DEPLOY_SIG, ADAPTER, adapterData.id, _encodeAdapterData(adapterData, baseAdapterData))
    );
    if (!success) revert UnderlyingError(returnData);

    adapter = abi.decode(returnData, (address));

    adminProxy.execute(adapter, abi.encodeWithSelector(IAdapter.setPerformanceFee.selector, performanceFee)); // call result not checked
  }
```

[L-02] When requiresInitData true, should check data.length is not zero

https://github.com/code-423n4/2023-01-popcorn/blob/36477d96788791ff07a1ba40d0c726fb39bf05ec/src/vault/CloneFactory.sol#L43

```solidity
if (template.requiresInitData) (success, ) = clone.call(data);
```

[L-03] Upgradeable contracts missing gap

https://github.com/code-423n4/2023-01-popcorn/blob/36477d96788791ff07a1ba40d0c726fb39bf05ec/src/vault/adapter/abstracts/OnlyStrategy.sol
https://github.com/code-423n4/2023-01-popcorn/blob/36477d96788791ff07a1ba40d0c726fb39bf05ec/src/utils/EIP165.sol
https://github.com/code-423n4/2023-01-popcorn/blob/36477d96788791ff07a1ba40d0c726fb39bf05ec/src/utils/OwnedUpgradeable.sol

Logic contracts require storage gaps included in the contract code to anticipate new state variables when a new logic implementation is deployed. The size of the gap needs to be updated appropriately after adding the new state variables.

[L-04] CloneRegistry addClone can add multiple clones by the same categry/template

https://github.com/code-423n4/2023-01-popcorn/blob/36477d96788791ff07a1ba40d0c726fb39bf05ec/src/vault/CloneRegistry.sol#L41-L51

```solidity
  function addClone(
    bytes32 templateCategory,
    bytes32 templateId,
    address clone
  ) external onlyOwner {
    cloneExists[clone] = true;
    clones[templateCategory][templateId].push(clone);
    allClones.push(clone);

    emit CloneAdded(clone);
  }
```

The addClone function in CloneRegistry should check for `cloneExists[clone] == false` before adding a new clone to the array.

Note that CloneRegistry getClonesByCategoryAndId and getAllClones can return an array with duplicate entries because of this.

[L-05] PermissionRegistry uses tricky whitelist vs blacklist flags

https://github.com/code-423n4/2023-01-popcorn/blob/36477d96788791ff07a1ba40d0c726fb39bf05ec/src/vault/PermissionRegistry.sol

https://github.com/code-423n4/2023-01-popcorn/blob/36477d96788791ff07a1ba40d0c726fb39bf05ec/src/vault/VaultController.sol#L704-L711

https://github.com/code-423n4/2023-01-popcorn/blob/36477d96788791ff07a1ba40d0c726fb39bf05ec/src/vault/VaultController.sol#L679-L689

In specific the presence of an endorsed address(0) affects the way tokens are either whitelisted or blacklisted, while the presence of an endorsed address(1) affects the way msg.senders are either whitelisted or blacklisted.
Since the addition of these two special addresses changes the way the permission registry is used to filter addresses, there should be a check that allows these two special addresses to be added only when there are no endorsed or rejected addresses in the regsitry already (except address(0) and address(1)) because it would change the meaning of the existing entries (for what concerns VaultControllers).

It would be preferable to use a WhiteListRegistry and/or BlackListRegistry as there could also be intersections between the set of addresses in the set controlled by address(0) and the set controlled by address(1) potentially causing more problems.

[L-06] All registries missing remove / maintenance functions (and GAS costs)

All registry contracts have functions to add entries, but no function to mantain or remove entries. While this simplifies the code it also poses a risk in the long term. Multiple places in the code indeed loops through the registries entries and make calls which in the long term may impact on the gas costs involved in certain functions.

[L-07] VaultController's verifyToken should check for address != 0

Depending on how the permissionRegistry is "configured" to work by the presence of the address(0) "flag", the _verifyToken function would allow a null token address to pass validation.

Add `if(token == address(0)) revert ZeroAddress();` at the beginning of the function.

https://github.com/code-423n4/2023-01-popcorn/blob/36477d96788791ff07a1ba40d0c726fb39bf05ec/src/vault/VaultController.sol#L679-L689

See the following code, where if endorsed(address(0)) the fail if permissionRegistry.endorsed(token) is not true (but token is zero, hence succeeds).

```solidity
  function _verifyToken(address token) internal view {
    if (
      (
        permissionRegistry.endorsed(address(0))
          ? !permissionRegistry.endorsed(token)
          : permissionRegistry.rejected(token)
      ) ||
      cloneRegistry.cloneExists(token) ||
      token == address(0)
    ) revert NotAllowed(token);
  }
```
