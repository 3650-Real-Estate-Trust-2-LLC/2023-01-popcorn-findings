## Summary
### Low-Risk Issues
|Number|Title|Context|
|:--:|:-------|:--:|
|[L-01]| Variable shadowing | 3 |
|[L-02]| ReentrancyGuardUpgradeable is not initialized | 1 |
|[L-03]| Missing zero address check for `nominatedOwner` | 2 |
|[L-04]| `OwnedUpgradeable` should inherit `Owner` instead of duplicating its code | 2 |
|[L-05]| Immutable variables are not `immutable` | 1 |

Total issues: 5

### Non-Critical Issues
|Number|Title|Context|
|:--:|:-------|:--:|
|[N-01]| Immutable instead of constant | 1 |
|[N-02]| Wrong/missing Natspec | 1 |
|[N-03]| Inconsistent function visibility order | 1 |
|[N-04]| Lock pragmas to a specific compiler version | 1 |
|[N-05]| Showing the full-length numbers in comments increase readability | 1 |

Total issues: 5

## Low Risk
### [L-01] Variable shadowing 

**Context:**

```solidity
xxx results - 3 files

src/utils/MultiRewardStaking.sol

- owner

128: if (caller != owner) _approve(owner, msg.sender, allowance(owner, msg.sender) - shares);
130: _burn(owner, shares);
133: emit Withdraw(caller, receiver, owner, assets, shares);
467: owner,
470: nonces[owner]++,
481: if (recoveredAddress == address(0) || recoveredAddress != owner) revert InvalidSigner(recoveredAddress);

src/vault/adapter/abstracts/AdapterBase.sol

- asset
- owner

src/vault/Vault.sol

- owner
```

**Recommendation**
Add an underscore to the variable (for example `_owner`) to avoid shadowing the storage variable.

### [L-02] ReentrancyGuardUpgradeable is not initialized

**Context**

```solidity
src/vault/adapter/abstracts/AdapterBase.sol

70: __Owned_init(_owner);
71: __Pausable_init();
72: __ERC4626_init(IERC20Metadata(asset));      
```

**Description**
AdapterBase extends `ReentrancyGuardUpgradeable`, but is missing `__ReentrancyGuard_init()` in the `__AdapterBase_init` function.

### [L-03] Missing zero address check for `nominatedOwner`

**Context**

```solidity
2 results - 2 files

src/utils/OwnedUpgradeable.sol

19: function nominateNewOwner(address _owner) external virtual onlyOwner {
20:    nominatedOwner = _owner;
21:    emit OwnerNominated(_owner);
22: }

src/utils/Owned.sol

17: function nominateNewOwner(address _owner) external virtual onlyOwner {
18:    nominatedOwner = _owner;
19:    emit OwnerNominated(_owner);
20: }
```

**Description**
Missing zero address check for new owners: even if it's not possible to transfer the ownership due to 2-step ownership, this could lead to a failed deploy due to misconfiguration.

### [L-04] `OwnedUpgradeable` should inherit `Owner` instead of duplicating its code

**Context**

```solidity
src/utils/OwnedUpgradeable.sol

src/utils/Owned.sol
```

### [L-05] Immutable variables are not immutable

**Context**

```solidity
src/vault/DeploymentController.sol

23: ICloneFactory public cloneFactory;
24: ICloneRegistry public cloneRegistry;
25: ITemplateRegistry public templateRegistry;
```

**Description**
There are some variables under the `IMMUTABLES` title that are not `immutable`: they could be overridden by inheriting contracts.

## Non-Critical

### [NC-01] Immutable instead of constant 

### [NC-02] Wrong/missing Natspec 

### [NC-03] Inconsistent function visibility order

### [NC-04] Lock pragmas to a specific compiler version

**Context**

```solidity
5 results - 4 files

2: pragma solidity ^0.8.15;

2: pragma solidity ^0.8.15;

2: pragma solidity ^0.8.15;

2: pragma solidity ^0.8.15;

```

**Description:**
Pragma statements are appropriate when the contract is a library that is intended for consumption by other developers. Otherwise, the developer would need to manually update the pragma, in order to compile locally.

[As a best practice](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/) it's better to lock the pragma to a specific version.

### [NC-05] Showing the full-length numbers in comments increase readability

```diff
src/vault/adapter/yearn/YearnAdapter.sol

- 25:      uint256 constant DEGRADATION_COEFFICIENT = 10**18;
+ 25:      uint256 constant DEGRADATION_COEFFICIENT = 10**18; // 1_000_000_000_000_000_000
```