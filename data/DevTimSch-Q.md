# [L-01] Anyone can let AdminProxy call arbitrary contracts
## Targets
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol
## Impact
There are no security checks in `VaultController.changeVaultFees(address[] calldata vaults)` and `VaultController.changeVaultAdapters(address[] calldata vaults)`. This means that anyone can call these functions with arbitrary addresses. This will let the AdminProxy execute a call on those addresses with `IVault.proposeFees.selector` or `IVault.changeAdapter.selector`. Low severity because I can't think of a critical attack vector through this, but should the adminProxy ever get more functions, this might cause trouble.

## Proof of Concept
Let's use the following contract as an example:

```solidity
pragma solidity ^0.8.15;

import {console} from "forge-std/console.sol";

contract AnyContract {

    function changeAdapter() external {
        console.log("Should never reach here");
    }
}
```

Now we can add a test case to `VaultController.t.sol`:

```solidity
    import {AnyContract} from "../AnyContract.sol";

    function test__callAnyContract() public {
        address[] memory targets = new address[](1);
        targets[0] = address(new AnyContract());
        controller.changeVaultAdapters(targets);
    }
```

Which should display the console message "Should never reach here".

## Tools Used
VS Code, Manual review, Foundry
## Recommended Mitigation Steps
Consider putting restrictions on who can call `changeAdapter` and `changeFees` in `VaultController`.
