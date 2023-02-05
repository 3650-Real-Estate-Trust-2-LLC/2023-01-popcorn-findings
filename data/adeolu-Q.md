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