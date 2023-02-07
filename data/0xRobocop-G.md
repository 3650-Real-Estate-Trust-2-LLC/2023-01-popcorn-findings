# [G-01] Sometimes is better to do assigments of structs from storage to local storage rather than doing them from storage to memory

The Solidity Documentation has a [section](https://docs.soliditylang.org/en/v0.8.18/types.html#data-location-and-assignment-behaviour) for data location and assigment behavior.  It says that when assigments are:

(1) From storage to memory it creates an independent copy
(2) From storage to local storage it only assigns a reference.

State variables that are structs can occupy more than 1 storage slot. When they are invoked during a function execution sometimes is better to do:

`Struct storage temp = TempMapping[key]` instead of `Struct memory temp = TempMapping[key]`.

The first one will only assign a reference, and you will use as many SLOADs instructions as many storage slots from the struct you read. The latter will do an independent copy, so that means that will SLOADs all the storage slots of the struct, even if you only read one during the function execution.

Instances found:

[LoC 254](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L254) at the MultiRewardStaking.sol contract.

# [G-02] Setting state variables to their default value refund gas.

At the page 12 of the [EVM yellowpaper](https://ethereum.github.io/yellowpaper/paper.pdf) one of the last paragraph says the following: 

"Storage fees have a slightly nuanced behaviour—to incentivise minimisation of the use of storage (which corresponds directly to a larger state database on all nodes), the execution fee for an operation that clears an entry in the storage is not only waived, a qualified refund is given; in fact, this refund is effectively paid up-front since the initial usage of a storage location costs substantially more than normal usage."

The `Vault.sol` contract in order to change `fees` and change the `adapter` uses a pattern similar to a 2-step transfer of ownership. The first transaction is used to propose the new values, and the second transaction confirm them. In the second transaction the proposed variables can be reset to their default values.

```
function changeAdapter() external takeFees {
        if (block.timestamp < proposedAdapterTime + quitPeriod)
            revert NotPassedQuitPeriod(quitPeriod);

        adapter.redeem(
            adapter.balanceOf(address(this)),
            address(this),
            address(this)
        );

        asset.approve(address(adapter), 0);

        emit ChangedAdapter(adapter, proposedAdapter);

        adapter = proposedAdapter;

        asset.approve(address(adapter), type(uint256).max);

        adapter.deposit(asset.balanceOf(address(this)), address(this));
    }

function changeFees() external {
        if (block.timestamp < proposedFeeTime + quitPeriod)
            revert NotPassedQuitPeriod(quitPeriod);

        emit ChangedFees(fees, proposedFees);
        fees = proposedFees;
    }
``` 

# [G-03] Don't repeat statements already executed on the same function.

The `fundReward` function at the [MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol) contract updates the `lastUpdatedTimestamp` of a staking token at the LoC 346:

`rewardInfos[rewardToken].lastUpdatedTimestamp = block.timestamp.safeCastTo32();`

But at the LoC 339 calls the internal function [_accrueRewards](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L339) that does the same.

# [G-04] The `_transfer(address, address, uint256)` internal function from the `MultiRewardStaking.sol` contract can be implemented in a more gas-efficient manner.

In order to apply the `_transfer` logic, it uses the internal functions `_burn` and `_mint` which are implemented the following way:

```
function _burn(address account, uint256 amount) internal virtual {
        require(account != address(0), "ERC20: burn from the zero address");

        _beforeTokenTransfer(account, address(0), amount);

        uint256 accountBalance = _balances[account];
        require(accountBalance >= amount, "ERC20: burn amount exceeds balance");
        unchecked {
            _balances[account] = accountBalance - amount;
            // Overflow not possible: amount <= accountBalance <= totalSupply.
            _totalSupply -= amount;
        }

        emit Transfer(account, address(0), amount);

        _afterTokenTransfer(account, address(0), amount);
    }

function _mint(address account, uint256 amount) internal virtual {
        require(account != address(0), "ERC20: mint to the zero address");

        _beforeTokenTransfer(address(0), account, amount);

        _totalSupply += amount;
        unchecked {
            // Overflow not possible: balance + amount is at most totalSupply + amount, which is checked above.
            _balances[account] += amount;
        }
        emit Transfer(address(0), account, amount);

        _afterTokenTransfer(address(0), account, amount);
    }
```
Logic that is repeated:

(1) The `_transfer` function validates at the beginning that both accounts are not the `address(0)`. Then again, both `_mint` and `_burn` checks the same. 
(2) The `_transfer` function validates that the `_from` account has the enough funds. The `_burn` makes the same validation.
(3) Both `_mint` and `_burn` modifies the `_totalSupply` state variable, this is necessary in the context of those functions. But in a transfer there is no need to modify the `_totalSupply` since it will be maintained.

I suggest to rewrite the function like this:

```
function _transfer(
    address from,
    address to,
    uint256 amount
  ) internal override accrueRewards(from, to) {
    super._transfer(from, to, amount);
 }
```
# [G-05] Use the `unchecked {}` block for arithmetic that wont overflow/underflow

Solidity from the version 0.8.0 added by default overflow and underflow checks, this increase the gas consumption. By wrapping the operations within `unchecked` blocks, the checks are disabled and the gas cost gets reduced.

The `claimRewards` function from the `MultiRewardEscrow.sol` contract is implemented as follows:

```
function claimRewards(bytes32[] memory escrowIds) external {
    for (uint256 i = 0; i < escrowIds.length; i++) {
      bytes32 escrowId = escrowIds[i];
      Escrow memory escrow = escrows[escrowId];

      uint256 claimable = _getClaimableAmount(escrow);
      if (claimable == 0) revert NotClaimable(escrowId);

      escrows[escrowId].balance -= claimable;
      escrows[escrowId].lastUpdateTime = block.timestamp.safeCastTo32();

      escrow.token.safeTransfer(escrow.account, claimable);
      emit RewardsClaimed(escrow.token, escrow.account, claimable);
    }
  }
```
The statement `escrows[escrowId].balance -= claimable;` can be wrapped on an `unchecked` block. This wont underflow because the `uint256 claimable` variable cannot be greater than `escrows[escrowId].balance ` since the function `_getClaimableAmount` returns the minimum number of two numbers, one being `escrow.balance`.

Other similar instances:

[LoC 102 MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L102)
[LoC 110 MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L110)

# [G-06] Use immutable variables when possible

Immutable variables don't use storage slots, their values are stored directly on the bytecode during deployment.

The variables: `ICloneFactory public cloneFactory;`, `ICloneRegistry public cloneRegistry;` and `ITemplateRegistry public templateRegistry;` of the `DeploymentController.sol` contract can be made immutable.
