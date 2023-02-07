# [NC-01] Inaccurate Natspec - The rewardsEndTimestamp cannot be zero

The `rewardsEndTimestamp` gets calculated based on `rewardsPerSecond` and `amount`. It is calculated and assigned the value in the function `addRewardToken`.
The minimum amount is `block.timestamp` at the time the reward token is added.
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L275-L277

Instance:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/IMultiRewardStaking.sol#L20


# [NC-02] Function state mutability can be restricted to view
There is no state change, the funcion should be change to view visibility

Instance:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L351

# [NC-03] NotEndorsed error is created but unused in CloneFactory
NotEndorsed error is created but not used however it was not used. 

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneFactory.sol#L32

Recommended Mitigation Steps: 

- Delete the unused variable or 
- Add logic to check if the template is endorsed or not before deploying

# [NC-04] Check if a clone exists before adding to `CloneRegistry` 

A clone should be checked if it exists to avoid duplication in `clones` and `allClones` arrays. Checking implementation in when adding template in registry
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L53


Instance:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneRegistry.sol#L41-L51

# [NC-05] Two different errors have the same custom Error

Input lengths mismatched and invalid Permission have the same custom error `Mismatch()`. 
They should have different error for easier debugging and logging.

Instance:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L40
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L43

# [L-01] Floating/Unlocked Pragma
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively or a newer complier version that the contracts are not tested.

pragma solidity ^0.8.15

Below instance for examples:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L4
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L3
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L4
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L4

# [L-02] isClaimable function misses case when block.timestamp < escrow.start
When `block.timestamp` < `escrow.start`, the escrow is locked and reward is not claimable. However the `isClaimable` function returns `true`.

## Instance:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L140-L142

## POC: 

// test/MultiRewardEscrow.t.sol

  function test__isClaimableWithCliff() public {
    vm.startPrank(alice);
    escrow.lock(iToken1, bob, 10 ether, 10, 10);
    bytes32[] memory bobEscrowIds = escrow.getEscrowIdsByUser(bob);
    assertFalse(escrow.isClaimable(bobEscrowIds[0]));
  }

## Recommended Mitigation Steps:

Add checking `block.timestamp > escrow.start` before return true.


# [L-03] Check if templateId is valid before setting it as the active Template

The input templateId is not checked to ensure that it is endorsed. Setting invalid templateId as active template could lead to fail in deploying vault and staking contract.

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L864



## Recommended Mitigation Steps:
Add checking if the input templateId is endorsed in templateRegistry