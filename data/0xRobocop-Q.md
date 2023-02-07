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

# [L-03] Is safer to cast `block.timestamp` to `uint64` instead of `uint32`
 
Instances:

[LoC 114 MultiRewardEscrow](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L114)
[LoC 163 MultiRewardEscrow](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L163)
[LoC 175 MultiRewardEscrow](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L175)