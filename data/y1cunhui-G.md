## Summary

| Index | Issue | Instances |
| -------- | -------- | -------- |
| [G-01]     | Use `unchecked` for arithmetic operations that could not overflow/underflow     | 3    |
| [G-02] | Variables only initialized in constructor should be marked as `immutable`| 3 |
| [G-03] | Use `a -= b` for storage variables can be expensive | 1 |
| [G-04] | Unnecessary complex arithmetic computation | 1 |
| [G-05] | `keccak256()` should only need to be called on a specific string literal once | 1 |
| [G-06] | `internal` function only used once can be inlined | 9|
| [G-07] | `accrueRewards` modifier iterate over all tokens each time | 1 |



## [G-01]  Use `unchecked` for arithmetic operations that could not overflow/underflow 

```solidity
File: src/utils/MultiRewardEscrow.sol
102    nonce++;
//nonce is uint256, so could not possibly overflow
```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L102

```solidity
File: src/utils/MultiRewardStaking.sol
392 if (rewards.rewardsEndTimestamp > block.timestamp) {
    //@audit unchecked this line, since lastUpdatedTimestamp cannot be greater then block.timestamp
      elapsed = block.timestamp - rewards.lastUpdatedTimestamp;
    } else if (rewards.rewardsEndTimestamp > rewards.lastUpdatedTimestamp) {
    //@audit uncheck this line, since if condition above
      elapsed = rewards.rewardsEndTimestamp - rewards.lastUpdatedTimestamp;
    }
```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L392

## [G-02]  Variables only initialized in constructor should be marked as `immutable`
```solidity
File: src/vault/DeploymentController.sol
      //@audit GAS-should be immutable
22    ICloneFactory public cloneFactory;
      ICloneRegistry public cloneRegistry;
      ITemplateRegistry public templateRegistry;
```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol

## [G-03]  Use `a -= b` for storage variables can be expensive
```solidity
File: src/utils/MultiRewardEscrow.sol
157   Escrow memory escrow = escrows[escrowId];
      ...
      //@audit 
      //should use "escrows[escrowId].balance = escrow.balance - claimable" 
      //to replace an SLOAD to MLOAD
162   escrows[escrowId].balance -= claimable;
      
```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L162

## [G-04]  Unnecessary complex arithmetic computation
```solidity
File: src/utils/MultiRewardEscrow.sol
179   return
      // @audit just compare block.timestamp and escrow.end instead
      // to avoid the complex computation each time 
      Math.min(
        (escrow.balance * (block.timestamp - uint256(escrow.lastUpdateTime))) /
          (uint256(escrow.end) - uint256(escrow.lastUpdateTime)),
        escrow.balance
      );

```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L179

## [G-05]  `keccak256()` should only need to be called on a specific string literal once
```solidity
File: src/utils/MultiRewardStaking.sol
       //@audit can be computed and saved as immutable
       // for this function is not `view`
466    keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),
```

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L466

## [G-06]  `internal` function only used once can be inlined
```solidity
File: src/utils/MultiRewardStaking.sol
function _lockToken(
    address user,
    IERC20 rewardToken,
    uint256 rewardAmount,
    EscrowInfo memory escrowInfo
) internal {}

```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L191 
```solidity
File: src/vault/VaultController.sol
function _deployVault(VaultInitParams memory vaultData, IDeploymentController _deploymentController)
    internal
    returns (address vault)

function _registerCreatedVault(
    address vault,
    address staking,
    VaultMetadata memory metadata
  ) internal

function _handleVaultStakingRewards(address vault, bytes memory rewardsData) internal

function __deployAdapter(
    DeploymentArgs memory adapterData,
    bytes memory baseAdapterData,
    IDeploymentController _deploymentController
  ) internal returns (address adapter)

function _encodeAdapterData(DeploymentArgs memory adapterData, bytes memory baseAdapterData)
    internal
    returns (bytes memory)

function _deployStrategy(DeploymentArgs memory strategyData, IDeploymentController _deploymentController)
    internal
    returns (address strategy)


function _registerVault(address vault, VaultMetadata memory metadata) internal 

function _verifyAdapterConfiguration(address adapter, bytes32 adapterId) internal view
```
## [G-07]  `accrueRewards` modifier iterate over all tokens each time

This modifier computes all rewards each time which is really gas-consuming. Consider add a parameter of time `IERC20[]` for considered tokens.
```solidity
File: src/utils/MultiRewardStaking.sol
371 modifier accrueRewards(address _caller, address _receiver) {
    IERC20[] memory _rewardTokens = rewardTokens;
    for (uint8 i; i < _rewardTokens.length; i++) {
      IERC20 rewardToken = _rewardTokens[i];
      RewardInfo memory rewards = rewardInfos[rewardToken];

      if (rewards.rewardsPerSecond > 0) _accrueRewards(rewardToken, _accrueStatic(rewards));
      _accrueUser(_receiver, rewardToken);

      // If a deposit/withdraw operation gets called for another user we should accrue for both of them to avoid potential issues like in the Convex-Vulnerability
      if (_receiver != _caller) _accrueUser(_caller, rewardToken);
    }
    _;
  }


// corresponding functions
function _deposit(
    address caller,
    address receiver,
    uint256 assets,
    uint256 shares
  ) internal override accrueRewards(caller, receiver)

function _withdraw(
    address caller,
    address receiver,
    address owner,
    uint256 assets,
    uint256 shares
  ) internal override accrueRewards(caller, receiver)

function _transfer(
    address from,
    address to,
    uint256 amount
  ) internal override accrueRewards(from, to)

function claimRewards(address user, IERC20[] memory _rewardTokens) external accrueRewards(msg.sender, user)
```

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L371