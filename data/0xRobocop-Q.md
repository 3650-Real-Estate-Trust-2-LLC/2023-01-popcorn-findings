# [L-01] Incorrect trigger of events can cause confusion on off-chain monitors.

The [_transfer](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L137) function of the `MultiRewardStaking.sol` contract emits three events of `Transfer`, one for the actual transfer and two more because it uses the internal functions `_burn` and `_mint` which also emits events of `Transfer`. 

Actual transfer `emit Transfer(from, to, amount);`
Mint `emit Transfer(address(0), to, amount);`
Burn `emit Transfer(from, address(0), amount);`

Off-chain monitors can get confused because of this. For example: Subtracting two times `amount` of tokens from `from` and adding two times `amount` to `to`.

# [L-02] Initializing multiple `MultiRewardStaking` contracts with the same `_stakingToken` can lead to confusion.

When a `MultiRewardStaking` contract is initialized its body of code contains:
```
_name = string(abi.encodePacked("Staked ", IERC20Metadata(address(_stakingToken)).name()));
_symbol = string(abi.encodePacked("pst-", IERC20Metadata(address(_stakingToken)).symbol()));
```
It writes to the `_name` and `_symbol` state variables based on the `_stakingToken` variable. If more than one staking contract uses the same `_stakingToken` they will have the same `_name` and `_symbol`, which can lead to confusion. 

There is another instance of this on the [initialize](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L57v) function of the `Vault.sol` contract.

# [L-03] Is safer to cast `block.timestamp` to `uint64` instead of `uint32`
 
Instances:

[LoC 114 MultiRewardEscrow](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L114)
[LoC 163 MultiRewardEscrow](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L163)
[LoC 175 MultiRewardEscrow](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L175)

# [L-04] Disable initializers on the implementations contracts.

Context: [Here](https://forum.openzeppelin.com/t/is-disableinitializers-necessary/31070)

Contracts that should disable initializers:

`Vault.sol`, `VaultController.sol`, `YearnAdapter.sol` and `BeefyAdapter`

# [L-05] The `Vault.sol` contract does not inherit the `IERC4626` interface

It is important to inherit the interface, so the compliance with the interface can be confirmed by the compiler, and also to add it to one of the supported interfaces (as EIP165).

# [N-01] Remove events or errors not used

Some instances found:

`event TemplateUpdated(bytes32, bytes32)` at the `TemplateRegistry.sol` contract.
`error NotEndorsed(bytes32)` at the `CloneFactory.sol` contract
`event NoFee()` at the `MultiRewardEscrow` contract

# [N-02] Some initialize functions of inherited upgreadable contract are not been called.

`Vault.sol` initialize contract does not call the initialize functions of the `Pausable` and `Reentrancy` contracts

# [N-03] NatSpec comments are incorrect.

DeploymentController.sol:
		LoC 60 it copied the comment for addTemplateCategory. Here is not a Category that is being added, but a Template.
		
	TemplateRegistry.sol:
		LoC 23 it should say 'DeploymentController' instead of AdminProxy.
		
	CloneRegistry.sol:
		LoC 21 it should say 'DeploymentController' instead of AdminProxy.
		
	CloneFactory.sol
		LoC 22 it should say 'DeploymentController' instead of AdminProxy.
