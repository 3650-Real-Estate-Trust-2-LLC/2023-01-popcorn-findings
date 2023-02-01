### 1st BUG
Cache Array Length Outside of Loop

#### Impact
Issue Information: 
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

Example
🤦 Bad:

for (uint256 i = 0; i < array.length; i++) {
    // invariant: array's length is not changed
}
🚀 Good:

uint256 len = array.length
for (uint256 i = 0; i < len; i++) {
    // invariant: array's length is not changed
}

#### Findings:
```
2023-01-popcorn/src/utils/MultiRewardEscrow.sol::52 => Escrow[] memory selectedEscrows = new Escrow[](escrowIds.length);
2023-01-popcorn/src/utils/MultiRewardEscrow.sol::198 => error ArraysNotMatching(uint256 length1, uint256 length2);
2023-01-popcorn/src/utils/MultiRewardEscrow.sol::208 => if (tokens.length != tokenFees.length) revert ArraysNotMatching(tokens.length, tokenFees.length);
2023-01-popcorn/src/vault/PermissionRegistry.sol::39 => uint256 len = targets.length;
2023-01-popcorn/src/vault/PermissionRegistry.sol::40 => if (len != newPermissions.length) revert Mismatch();
2023-01-popcorn/src/vault/VaultController.sol::112 => if (rewardsData.length > 0) _handleVaultStakingRewards(vault, rewardsData);
2023-01-popcorn/src/vault/VaultController.sol::314 => uint8 len = uint8(vaults.length);
2023-01-popcorn/src/vault/VaultController.sol::316 => _verifyEqualArrayLength(len, newAdapter.length);
2023-01-popcorn/src/vault/VaultController.sol::336 => uint8 len = uint8(vaults.length);
2023-01-popcorn/src/vault/VaultController.sol::353 => uint8 len = uint8(vaults.length);
2023-01-popcorn/src/vault/VaultController.sol::355 => _verifyEqualArrayLength(len, fees.length);
2023-01-popcorn/src/vault/VaultController.sol::373 => uint8 len = uint8(vaults.length);
2023-01-popcorn/src/vault/VaultController.sol::434 => _verifyEqualArrayLength(vaults.length, rewardTokenData.length);
2023-01-popcorn/src/vault/VaultController.sol::436 => uint8 len = uint8(vaults.length);
2023-01-popcorn/src/vault/VaultController.sol::488 => uint8 len = uint8(vaults.length);
2023-01-popcorn/src/vault/VaultController.sol::490 => _verifyEqualArrayLength(len, rewardTokens.length);
2023-01-popcorn/src/vault/VaultController.sol::491 => _verifyEqualArrayLength(len, rewardsSpeeds.length);
2023-01-popcorn/src/vault/VaultController.sol::517 => uint8 len = uint8(vaults.length);
2023-01-popcorn/src/vault/VaultController.sol::519 => _verifyEqualArrayLength(len, rewardTokens.length);
2023-01-popcorn/src/vault/VaultController.sol::520 => _verifyEqualArrayLength(len, amounts.length);
2023-01-popcorn/src/vault/VaultController.sol::544 => _verifyEqualArrayLength(tokens.length, fees.length);
2023-01-popcorn/src/vault/VaultController.sol::563 => uint8 len = uint8(templateCategories.length);
2023-01-popcorn/src/vault/VaultController.sol::583 => uint8 len = uint8(templateCategories.length);
2023-01-popcorn/src/vault/VaultController.sol::584 => _verifyEqualArrayLength(len, templateIds.length);
2023-01-popcorn/src/vault/VaultController.sol::606 => uint8 len = uint8(vaults.length);
2023-01-popcorn/src/vault/VaultController.sol::619 => uint8 len = uint8(vaults.length);
2023-01-popcorn/src/vault/VaultController.sol::632 => uint8 len = uint8(vaults.length);
2023-01-popcorn/src/vault/VaultController.sol::645 => uint8 len = uint8(vaults.length);
2023-01-popcorn/src/vault/VaultController.sol::700 => function _verifyEqualArrayLength(uint256 length1, uint256 length2) internal pure {
2023-01-popcorn/src/vault/VaultController.sol::701 => if (length1 != length2) revert ArrayLengthMissmatch();
2023-01-popcorn/src/vault/VaultController.sol::765 => uint8 len = uint8(adapters.length);
2023-01-popcorn/src/vault/VaultController.sol::805 => uint8 len = uint8(adapters.length);
2023-01-popcorn/src/vault/VaultRegistry.sol::68 => return allVaults.length;
```
#### Tools used
Manual VS code review

### 2nd BUG
Use immutable for OpenZeppelin AccessControl's Roles Declarations

#### Impact
Issue Information: 
⚡️ Only valid for solidity versions <0.6.12 ⚡️

Access roles marked as constant results in computing the keccak256 operation each time the variable is used because assigned operations for constant variables are re-evaluated every time.

Changing the variables to immutable results in computing the hash only once on deployment, leading to gas savings.

Example
🤦 Bad:

bytes32 public constant GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");
🚀 Good:

bytes32 public immutable GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");

#### Findings:
```
2023-01-popcorn/src/utils/MultiRewardStaking.sol::460 => keccak256(
2023-01-popcorn/src/utils/MultiRewardStaking.sol::464 => keccak256(
2023-01-popcorn/src/utils/MultiRewardStaking.sol::466 => keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),
2023-01-popcorn/src/utils/MultiRewardStaking.sol::493 => keccak256(
2023-01-popcorn/src/utils/MultiRewardStaking.sol::495 => keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
2023-01-popcorn/src/utils/MultiRewardStaking.sol::496 => keccak256(bytes(name())),
2023-01-popcorn/src/utils/MultiRewardStaking.sol::497 => keccak256("1"),
2023-01-popcorn/src/vault/Vault.sol::93 => contractName = keccak256(
2023-01-popcorn/src/vault/Vault.sol::679 => keccak256(
2023-01-popcorn/src/vault/Vault.sol::683 => keccak256(
2023-01-popcorn/src/vault/Vault.sol::685 => keccak256(
2023-01-popcorn/src/vault/Vault.sol::718 => keccak256(
2023-01-popcorn/src/vault/Vault.sol::720 => keccak256(
2023-01-popcorn/src/vault/Vault.sol::723 => keccak256(bytes(name())),
2023-01-popcorn/src/vault/Vault.sol::724 => keccak256("1"),
2023-01-popcorn/src/vault/VaultController.sol::40 => bytes4 internal immutable DEPLOY_SIG = bytes4(keccak256("deploy(bytes32,bytes32,bytes)"));
2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol::647 => keccak256(
2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol::651 => keccak256(
2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol::653 => keccak256(
2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol::686 => keccak256(
2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol::688 => keccak256(
2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol::691 => keccak256(bytes(name())),
2023-01-popcorn/src/vault/adapter/abstracts/AdapterBase.sol::692 => keccak256("1"),
```
#### Tools used
Manual VS code review

### 3rd BUG
Long Revert Strings

#### Impact
Issue Information: 
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.

Example
🤦 Bad:

require(condition, "UniswapV3: The reentrancy guard. A transaction cannot re-enter the pool mid-swap");
🚀 Good (with shorter string):

// TODO: Provide link to a reference of error codes
require(condition, "LOK");
🚀 Good (with custom errors):

/// @notice A transaction cannot re-enter the pool mid-swap.
error NoReentrancy();

// ...

if (!condition) {
    revert NoReentrancy();
}

#### Findings:
```
2023-01-popcorn/src/utils/MultiRewardStaking.sol::466 => keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),
2023-01-popcorn/src/utils/MultiRewardStaking.sol::495 => keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
2023-01-popcorn/src/utils/Owned.sol::23 => require(msg.sender == nominatedOwner, "You must be nominated before you can accept ownership");
2023-01-popcorn/src/utils/Owned.sol::35 => require(msg.sender == owner, "Only the contract owner may perform this action");
2023-01-popcorn/src/utils/OwnedUpgradeable.sol::25 => require(msg.sender == nominatedOwner, "You must be nominated before you can accept ownership");
2023-01-popcorn/src/utils/OwnedUpgradeable.sol::37 => require(msg.sender == owner, "Only the contract owner may perform this action");
2023-01-popcorn/src/vault/Vault.sol::94 => abi.encodePacked("Popcorn", name(), block.timestamp, "Vault")
```
#### Tools used
Manual VS code review