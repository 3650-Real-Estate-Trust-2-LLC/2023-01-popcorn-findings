# QA Report for Popcorn contest
## Overview
During the audit, 1 low and 8 non-critical issues were found.

№ | Title | Risk Rating  | Instance Count
--- | --- | --- | ---
L-1 | No way to remove elements from ```rewardTokens``` array | Low | 1
NC-1 | Order of Functions | Non-Critical | 16
NC-2 | Order of Layout | Non-Critical | 5
NC-3 | Visibility is not set | Non-Critical | 2
NC-4 | Typos | Non-Critical | 6
NC-5 | Missing leading underscores | Non-Critical | 5
NC-6 | Unused variable | Non-Critical | 2
NC-7 | Missing NatSpec | Non-Critical | 18
NC-8 | Maximum line length exceeded | Non-Critical | 17

## Low Risk Findings(1)
### L-1. No way to remove elements from ```rewardTokens``` array
##### Description
There is no way to remove items from the [```rewardTokens```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L208) array.  Over time, some rewardTokens may become irrelevant, or, accidentally, an incorrect rewardToken may be added to the array.  In the best case, all functions that use the [```accrueRewards```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L371-L384) modifier will consume more gas, and in the worst case, due to one small mistake, all these functions may become unavailable.
##### Instances
- [```IERC20[] public rewardTokens;```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L208)
- [```rewardTokens.push(rewardToken);```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L263) (Elements can only be added, not removed)
- [```modifier accrueRewards(address _caller, address _receiver) {```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L371) (The modifier loops through the [```rewardTokens```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L372-L373) array)

Functions affected by modifier [```accrueRewards```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L371) :
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L112
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L127
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L141
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L170

##### Recommendation
Implement function for deleting elements from the [```rewardTokens```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L208) array.
#
## Non-Critical Risk Findings(8)
### NC-1. Order of Functions
##### Description
According to [Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-functions), ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.  
Functions should be grouped according to their visibility and ordered:
1) constructor
2) receive function (if exists)
3) fallback function (if exists)
4) external
5) public
6) internal
7) private
##### Instances
Public functions should be placed before internal:
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L129
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L122

External function should be placed before internal:
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L362


Public functions should be placed after external:
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L124-L158 
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L170-L240
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L285-L350
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L399
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L433-L474

Public functions should be placed before internal:
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L173-L204 
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L271-L287
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L304
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L329
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L353

It is better to place initialize() function before other functions:
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IAdapter.sol#L34
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L52
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVault.sol#L90

##### Recommendation
Reorder functions where possible.
#
### NC-2. Order of Layout
##### Description
According to [Order of Layout](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout), inside each contract, library or interface, use the following order:
1) Type declarations
2) State variables
3) Events
4) Modifiers
5) Functions
##### Instances
Modifiers should be placed before functions:
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L371
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L480
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L496
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L704
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L559

##### Recommendation
Place modifiers before functions.
#
### NC-3. Visibility is not set
##### Instances
- [```uint256 constant DEGRADATION_COEFFICIENT = 10**18;```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L25) 
- [```uint256 constant SECONDS_PER_YEAR = 365.25 days;```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L35) 

##### Recommendation
It is better to specify visibility explicitly.
#
### NC-4. Typos
##### Instances
- [```* Is used by `VaultController` to check if a target is a registerd clone.```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L14) => ```registered```
- [```* @param template Contains the implementation address and necessary informations to clone the implementation.```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L65) => ```information```
- [```/// @notice The amount of assets that are free to be withdrawn from the yVault after locked profts.```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L100) => ```profits```
- [```* @dev This function is the one stop solution to create a new vault with all necessary admin functions or auxiliery contracts.```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L87) => ```auxiliary```
- [```* @notice Sets a new `DeploymentController` and saves its auxilary contracts. Caller must be owner.```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L829) => ```auxiliary```
- [```/// @Notice Optional - Metadata CID which can be used by the frontend to add informations to a vault/adapter...```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ITemplateRegistry.sol#L13) => ```information```

#
### NC-5. Missing leading underscores
##### Description
Internal constants and state variables should have a leading underscore.
##### Instances
- [```uint256 constant DEGRADATION_COEFFICIENT = 10**18;```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L25) 
- [```uint256 internal nonce;```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L71) 
- [```uint256 constant SECONDS_PER_YEAR = 365.25 days;```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L35) 
- [```bytes4 internal immutable DEPLOY_SIG = bytes4(keccak256("deploy(bytes32,bytes32,bytes)"));```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L40) 
- [```uint256 internal underlyingBalance;```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L241) 

##### Recommendation
Add leading underscores where needed.
#
### NC-6. Unused variable
##### Description
Some variables declared but not used.
##### Instances
- [```uint256 highWaterMark_ = highWaterMark;```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L448) (highWaterMark_)
- [```function _registerVault(address vault, VaultMetadata memory metadata) internal {```](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L390) (vault)

#
### NC-7. Missing NatSpec
##### Description
NatSpec is missing for 18 functions in 6 contracts.
##### Instances
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/OnlyStrategy.sol#L10
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L84 
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L158
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L166
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L140
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L144
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L170
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L108
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L117
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L351
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L124
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L160
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L200
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L242
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L496
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L164 
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L704
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L836

##### Recommendation
Add NatSpec for all functions.
#
### NC-8. Maximum line length exceeded
##### Description
According to [Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#maximum-line-length), maximum suggested line length is 120 characters.  Longer lines make the code harder to read.
##### Instances
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L13
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultRegistry.sol#L41
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L91
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L49
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L49
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L22
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L291
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L380
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L55
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L398
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L408
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L425
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L426
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L87
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L477
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L604
- https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L630

##### Recommendation
Make the lines shorter.