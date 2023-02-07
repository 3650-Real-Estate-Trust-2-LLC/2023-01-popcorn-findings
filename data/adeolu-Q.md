## Address(0) and address(this) can be saved as clone addresses to clone registry
It is possible to add address(0) and address(this) to the cloneRegistry contract's allClones mapping. There should be checks to prevent this. 

**Link to affected part of code** -->  https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneRegistry.sol#L41


**Proof of concept code**
```
// SPDX-License-Identifier: GPL-3.0
// Docgen-SOLC: 0.8.15

pragma solidity ^0.8.15;

import { Test } from "forge-std/Test.sol";
import { CloneRegistry } from "../../src/vault/CloneRegistry.sol";

contract CloneRegistryTest is Test {
  CloneRegistry registry;

  bytes32 templateCategory = "templateCategory";
  bytes32 templateId = "templateId";

  event CloneAdded(address clone);

  function setUp() public {
    registry = new CloneRegistry(address(this));
  }

  /*//////////////////////////////////////////////////////////////
                              ADD CLONE
    //////////////////////////////////////////////////////////////*/

  function test__addCloneAsRegistry() public {
    assertEq(registry.cloneExists(address(registry)), false);

    vm.expectEmit(false, false, false, true);
    emit CloneAdded(address(registry));

    registry.addClone(templateCategory, templateId, address(registry));

    assertEq(registry.cloneExists(address(registry)), true);
    assertEq(registry.clones(templateCategory, templateId, 0), address(registry));
    assertEq(registry.allClones(0), address(registry));
  }

    function test__addCloneAsAddr0() public {
    assertEq(registry.cloneExists(address(0)), false);

    vm.expectEmit(false, false, false, true);
    emit CloneAdded(address(0));

    registry.addClone(templateCategory, templateId, address(0));

    assertEq(registry.cloneExists(address(0)), true);
    assertEq(registry.clones(templateCategory, templateId, 0), address(0));
    assertEq(registry.allClones(0), address(0));
  }
}
```
**Recommended mitigation** ---> add a check in the addClone function that reverts when address(0) is supplied as clone address.


## Event  PermissionSet should be emitted after the permission is actually set

**Link to code in repo** --> https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L45


```
  function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {
    uint256 len = targets.length;
    if (len != newPermissions.length) revert Mismatch();

    for (uint256 i = 0; i < len; i++) {
      if (newPermissions[i].endorsed && newPermissions[i].rejected) revert Mismatch();

      emit PermissionSet(targets[i], newPermissions[i].endorsed, newPermissions[i].rejected);

      permissions[targets[i]] = newPermissions[i];
    }
  }
```
**Recommended mitigation** -->  The code above emits the PermissionSet event before the permission is actually set in logic. The event PermissionSet should be emitted after the permission is actually set so in this function the event emit should be the last line of code logic here, like this;

```
  function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {
    uint256 len = targets.length;
    if (len != newPermissions.length) revert Mismatch();

    for (uint256 i = 0; i < len; i++) {
      if (newPermissions[i].endorsed && newPermissions[i].rejected) revert Mismatch();

      permissions[targets[i]] = newPermissions[i];

     emit PermissionSet(targets[i], newPermissions[i].endorsed, newPermissions[i].rejected);
    }
  }
```


## Emit event ChangedFees after fees has really been changed in logic

**link to affected code** --> https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L544

```
    /// @notice Change fees to the previously proposed fees after the quit period has passed.
    function changeFees() external {
        if (block.timestamp < proposedFeeTime + quitPeriod)
            revert NotPassedQuitPeriod(quitPeriod);

        emit ChangedFees(fees, proposedFees);
        fees = proposedFees;
    }
```

In the above function event is emitted before the fees value is actually changed in storage.

**Recommended Mitigation** --> emit event after the fees value is set to the  proposedFees.


## Emit event FeeRecipientUpdated after feeReceipient has really been changed in logic

**link to affected code** --> https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L553

```
    function setFeeRecipient(address _feeRecipient) external onlyOwner {
        if (_feeRecipient == address(0)) revert InvalidFeeRecipient();

        emit FeeRecipientUpdated(feeRecipient, _feeRecipient);

        feeRecipient = _feeRecipient;
    }
```

In the above function event is emitted before the fees value is actually changed in storage.

**Recommended Mitigation** --> emit event after the fees value is set to the  proposedFees.


## getSubmitter() and getVault() return same object

**Links to affected code in repo** --> https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L59, https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L75

```
  function getVault(address vault) external view returns (VaultMetadata memory) {
    return metadata[vault];
  }

  function getSubmitter(address vault) external view returns (VaultMetadata memory) {
    return metadata[vault];
  }
```

**Recommended Mitigation** ---> The above functions are both in the VaultRegistry contract and they return the exact same thing, even though they are named differently. Perhaps getSubmitter is to return the `creator` value in `VaultMetadata` struct? If this is the case the code in getSubmitter() should be updated to return such (metadata[vault].creator) or one of the functions should be removed to reduce bytecode size as they are both doing the same thing.



## multiple checking for equal array length in setEscrowTokenFees() in VaultController contract 

```
  function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {
    _verifyEqualArrayLength(tokens.length, fees.length);
    (bool success, bytes memory returnData) = adminProxy.execute(
      address(escrow),
      abi.encodeWithSelector(IMultiRewardEscrow.setFees.selector, tokens, fees)
    );
    if (!success) revert UnderlyingError(returnData);
  }
```
link to setEscrowTokenFees in repo --> https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L543

In `setEscrowTokenFees` in `VaultController` contract, the lengths of  token and fees arrays are checked to be equal and it is also checked for equality again in the `setFees` function in `MultiRewardEscrow` contract. See setFees function in MultiRewardEscrow below;

```
  function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {
    if (tokens.length != tokenFees.length) revert ArraysNotMatching(tokens.length, tokenFees.length);

    for (uint256 i = 0; i < tokens.length; i++) {
      if (tokenFees[i] >= 1e17) revert DontGetGreedy(tokenFees[i]);

      fees[tokens[i]].feePerc = tokenFees[i];
      emit FeeSet(tokens[i], tokenFees[i]);
    }
  }
```
link to setFees  in repo ---> https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L207

**Recommended mitigation** ---> Remove the check in one of the two functions, I suggest removing the equal length check in `setEscrowTokenFees` as `setFees` is the underlying function being called in `setEscrowTokenFees`



## Tools used
VS Code, Forge
