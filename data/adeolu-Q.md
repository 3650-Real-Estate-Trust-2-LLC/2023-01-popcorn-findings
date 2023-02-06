## Address(0) and address(this) can be saved as clone addresses to clone registry
It is possible to add address(0) and address(this) to the cloneRegistry contract's allClones mapping. There should be checks to prevent this. 

Link to affected part of code -->  https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneRegistry.sol#L41


Proof of concept code 
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



## Event  PermissionSet should be emitted after the permission is actually set

Link to code in repo --> https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L45


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
The code above emits the PermissionSet event before the permission is actually set in logic. The event PermissionSet should be emitted after the permission is actually set so in this function the event emit should be the last line of code logic here, like this;

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
