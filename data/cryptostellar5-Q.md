
### 01 MISSING EVENTS FOR FUNCTIONS THAT CHANGE CRITICAL PARAMETERS 

*Number of Instances Identified: 4*

The functions that change critical parameters should emit events. Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services. The alternative of directly querying on-chain contract state for such changes is not considered practical for most users/usages.

### Recommended Mitigation Steps

Add events to all functions that change critical parameters.


https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol

```
408: function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {
543: function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {
764: function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {
804: function setAdapterHarvestCooldowns(address[] calldata adapters) external onlyOwner {
```



### 02 ADDING A RETURN STATEMENT WHEN THE FUNCTION DEFINES A NAMED RETURN VARIABLE, IS REDUNDANT

*Number of Instances Identified: 2*

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol

```
19-22: function execute(address target, bytes calldata callData)
    external
    onlyOwner
    returns (bool success, bytes memory returndata)
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol

```
380-385: function _convertToShares(uint256 assets, Math.Rounding rounding)
        internal
        view
        virtual
        override
        returns (uint256 shares)
```




### 03  PREVENT SIGNATURE MALLEABILITY

*Number of Instances Identified: 3*

Use OpenZeppelin's `ECDSA` contract rather than calling `ecrecover()` directly


https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol

```
459: address recoveredAddress = ecrecover(
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol

```
678: address recoveredAddress = ecrecover(
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol

```
646: address recoveredAddress = ecrecover(
```




### 04 USE SCIENTIFIC NOTATION INSTEAD OF EXPONENTITATION

*Number of Instances Identified: 3*

Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. `10**18`)

Scientific notation should be used for better code readability

_There are 3 instances of this issue_

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol

```
274: uint64 ONE = (10**IERC20Metadata(address(rewardToken)).decimals()).safeCastTo64();
406: deltaIndex = accrued.mulDiv(uint256(10**decimals()), supplyTokens, Math.Rounding.Down).safeCastTo224();
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol

```
25: uint256 constant DEGRADATION_COEFFICIENT = 10**18;
```


### 05 FLOATING PRAGMA VERSION SHOULD NOT BE USED

*Number of Instances Identified: 35*

In the contracts, floating pragmas should not be used. Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

[https://swcregistry.io/docs/SWC-103](https://swcregistry.io/docs/SWC-103)

### Proof of Concept

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/EIP165.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/OnlyStrategy.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/WithRewards.sol#L4 
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneFactory.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultRegistry.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L3
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IEIP165.sol#L2
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IAdminProxy.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IWithRewards.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ICloneFactory.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IStrategy.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ICloneRegistry.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IPermissionRegistry.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVaultRegistry.sol#L3
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/IYearn.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IAdapter.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IDeploymentController.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ITemplateRegistry.sol#L3
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardEscrow.sol#L3
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IERC4626.sol#L3
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/IBeefy.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L3
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVault.sol#L3
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVaultController.sol#L4



### 06 OPEN TODOS

*Number of Instances Identified: 1*

Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol

```
516: // TODO use deterministic fee recipient proxy
```



### 07 TYPOs

_There are 2 instances of this issue_

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol

```
87: * @dev This function is the one stop solution to create a new vault with all necessary admin functions or auxiliery contracts. //@audit auxiliary instead of auxiliery

181: * @notice Deploy a new Adapter with our without a strategy. Caller must be owner. //@audit  or instead of our
```



### 08 USE LATEST SOLIDITY VERSION

*Number of Instances Identified: 35*

When deploying contracts, you should use the latest released version of Solidity. Apart from exceptional cases, only the latest version receives [security fixes](https://github.com/ethereum/solidity/security/policy#supported-versions). Furthermore, breaking changes as well as new features are introduced regularly.
Latest Version is 0.8.17

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/EIP165.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/OnlyStrategy.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/WithRewards.sol#L4 
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneFactory.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultRegistry.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L3
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IEIP165.sol#L2
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IAdminProxy.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IWithRewards.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ICloneFactory.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IStrategy.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ICloneRegistry.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IPermissionRegistry.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVaultRegistry.sol#L3
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/IYearn.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IAdapter.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IDeploymentController.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ITemplateRegistry.sol#L3
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardEscrow.sol#L3
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IERC4626.sol#L3
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/IBeefy.sol#L4
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L3
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVault.sol#L3
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVaultController.sol#L4

