
## 01 safeapprove() is deprecated

[Deprecated](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/bfff03c0d2a59bcd8e2ead1da9aed9edf0080d05/contracts/token/ERC20/utils/SafeERC20.sol#L38-L45) in favor of `safeIncreaseAllowance()` and `safeDecreaseAllowance()`. If only setting the initial allowance to the value that means infinite, `safeIncreaseAllowance()` can be used instead

_There is 1 instance of this issue:_

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol

```
File: src/utils/MultiRewardStaking.sol

271: rewardToken.safeApprove(address(escrow), type(uint256).max);
```

----

## 02 Best practice is to prevent signature malleability

Use OpenZeppelin’s `ECDSA` contract rather than calling `ecrecover()` directly

_There are 3 instances of this issue_

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol

```
File: src/utils/MultiRewardStaking.sol

459: address recoveredAddress = ecrecover(
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol

```
File: src/vault/Vault.sol

678: address recoveredAddress = ecrecover(
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol

```
File: src/vault/adapter/abstracts/AdapterBase.sol

646: address recoveredAddress = ecrecover(
```

------

## 03 Missing events for functions that change critical parameters 

The afunctions that change critical parameters should emit events. Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services. The alternative of directly querying on-chain contract state for such changes is not considered practical for most users/usages.

Missing events and timelocks do not promote transparency and if such changes immediately affect users’ perception of fairness or trustworthiness, they could exit the protocol causing a reduction in liquidity which could negatively impact protocol TVL and reputation.

### Recommended Mitigation Steps

Add events to all functions that change critical parameters.

_There are 4 instances of this issue_

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol

```
File: src/vault/VaultController.sol

408: function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {
543: function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {
764: function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {
804: function setAdapterHarvestCooldowns(address[] calldata adapters) external onlyOwner {
```

----

## 04 Adding a return statement when the function defines a named return variable, is redundant

_There are 2 instances of this issue_

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol

```
File: src/vault/AdminProxy.sol

19-22: function execute(address target, bytes calldata callData)
    external
    onlyOwner
    returns (bool success, bytes memory returndata)
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol

```
File: src/vault/adapter/abstracts/AdapterBase.sol

380-385: function _convertToShares(uint256 assets, Math.Rounding rounding)
        internal
        view
        virtual
        override
        returns (uint256 shares)
```

-----

## 05 Use scientific notation rather than exponentiation

Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. `10**18`)

Scientific notation should be used for better code readability

_There are 3 instances of this issue_

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol

```
File: src/utils/MultiRewardStaking.sol

274: uint64 ONE = (10**IERC20Metadata(address(rewardToken)).decimals()).safeCastTo64();
406: deltaIndex = accrued.mulDiv(uint256(10**decimals()), supplyTokens, Math.Rounding.Down).safeCastTo224();
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol

```
File: src/vault/adapter/yearn/YearnAdapter.sol

25: uint256 constant DEGRADATION_COEFFICIENT = 10**18;
```

-----------

## 06 Open TODOS

Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment

_There is 1 instance of this issue_

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol

```
File: src/vault/adapter/abstracts/AdapterBase.sol

516: // TODO use deterministic fee recipient proxy
```

-------

## 07 Typos

_There are 3 instances of this issue_

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ITemplateRegistry.sol

```
File: src/interfaces/vault/ITemplateRegistry.sol

13: /// @Notice Optional - Metadata CID which can be used by the frontend to add informations to a vault/adapter...
```

Typo: It should be `information` instead of `informations`

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol

```
File: src/vault/VaultController.sol

87: * @dev This function is the one stop solution to create a new vault with all necessary admin functions or auxiliery contracts.
181: * @notice Deploy a new Adapter with our without a strategy. Caller must be owner.
```

Typo: In line 87, it should be `auxiliary` instead of `auxiliery`
In line 181, it should be `or` instead of `our`

-------

## 08 Use a more recent version of solidity

It's a best practice to use the latest compiler version.
Older compilers might be susceptible to some bugs. We recommend changing the solidity version pragma to the latest version to enforce the use of an up to date compiler.

List of known compiler bugs and their severity can be found here: [https://etherscan.io/solcbuginfo](https://etherscan.io/solcbuginfo)

_This issue exists in all the In-scope contracts_. _There are 35 instances of this issue_

[EIP165.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/EIP165.sol#L4) , [OnlyStrategy.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/OnlyStrategy.sol#L4) , [AdminProxy.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L4) , [WithRewards.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/WithRewards.sol#L4) , [CloneFactory.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneFactory.sol#L4) , [PermissionRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L4) , [CloneRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L4) , [VaultRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultRegistry.sol#L4) , [DeploymentController.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L4) , [TemplateRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L4) , [YearnAdapter.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L4) , [MultiRewardEscrow.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L4) , [BeefyAdapter.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L4) , [MultiRewardStaking.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L4) , [Vault.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L4) , [VaultController.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L3) , [AdapterBase.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L4) , [IEIP165.sol#L2](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IEIP165.sol#L2) , [IAdminProxy.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IAdminProxy.sol#L4) , [IWithRewards.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IWithRewards.sol#L4) , [ICloneFactory.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ICloneFactory.sol#L4) , [IStrategy.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IStrategy.sol#L4) , [ICloneRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ICloneRegistry.sol#L4) , [IPermissionRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IPermissionRegistry.sol#L4) , [IVaultRegistry.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVaultRegistry.sol#L3) , [IYearn.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/IYearn.sol#L4) , [IAdapter.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IAdapter.sol#L4) , [IDeploymentController.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IDeploymentController.sol#L4) , [ITemplateRegistry.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ITemplateRegistry.sol#L3) , [IMultiRewardEscrow.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardEscrow.sol#L3) , [IERC4626.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IERC4626.sol#L3) , [IBeefy.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/IBeefy.sol#L4) , [IMultiRewardStaking.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L3) , [IVault.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVault.sol#L3) , [IVaultController.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVaultController.sol#L4)

----------

## 09 Non-library or interface files should use fixed compiler versions, not floating ones

In the contracts, floating pragmas should not be used. Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

_This issue exists in all the In-scope contracts_. _There are 35 instances of this issue_

[EIP165.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/EIP165.sol#L4) , [OnlyStrategy.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/OnlyStrategy.sol#L4) , [AdminProxy.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L4) , [WithRewards.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/WithRewards.sol#L4) , [CloneFactory.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneFactory.sol#L4) , [PermissionRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L4) , [CloneRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L4) , [VaultRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultRegistry.sol#L4) , [DeploymentController.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L4) , [TemplateRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L4) , [YearnAdapter.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L4) , [MultiRewardEscrow.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L4) , [BeefyAdapter.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L4) , [MultiRewardStaking.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L4) , [Vault.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L4) , [VaultController.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L3) , [AdapterBase.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L4) , [IEIP165.sol#L2](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IEIP165.sol#L2) , [IAdminProxy.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IAdminProxy.sol#L4) , [IWithRewards.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IWithRewards.sol#L4) , [ICloneFactory.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ICloneFactory.sol#L4) , [IStrategy.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IStrategy.sol#L4) , [ICloneRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ICloneRegistry.sol#L4) , [IPermissionRegistry.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IPermissionRegistry.sol#L4) , [IVaultRegistry.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVaultRegistry.sol#L3) , [IYearn.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/IYearn.sol#L4) , [IAdapter.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IAdapter.sol#L4) , [IDeploymentController.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IDeploymentController.sol#L4) , [ITemplateRegistry.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ITemplateRegistry.sol#L3) , [IMultiRewardEscrow.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardEscrow.sol#L3) , [IERC4626.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IERC4626.sol#L3) , [IBeefy.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/IBeefy.sol#L4) , [IMultiRewardStaking.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L3) , [IVault.sol#L3](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVault.sol#L3) , [IVaultController.sol#L4](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVaultController.sol#L4)

-----------
