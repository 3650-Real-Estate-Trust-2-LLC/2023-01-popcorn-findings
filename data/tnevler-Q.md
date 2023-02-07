# Report
## Non-Critical Issues ##

### [N-1]: Function defines a named return variable but then it uses return statements
**Context:**

1. ```return target.call(callData);``` [L24](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L24) 

**Recommendation:**

Choose named return variable or return statement. It is unnecessary to use both.

### [N-2]: Variable is unused
**Context:**

1. ```function _registerVault(address vault, VaultMetadata memory metadata) internal {``` [L390](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L390) 

### [N-3]: Wrong order of functions
**Context:**

1. ```event Deployment(address indexed clone);``` [L29](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneFactory.sol#L29) (event definition can not go after constructor)
1. ```mapping(address => Permission) public permissions;``` [L26](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L26) (state variable declaration can not go after constructor)
1. ```mapping(address => bool) public cloneExists;``` [L28](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L28) (state variable declaration can not go after constructor)
1. ```mapping(address => VaultMetadata) public metadata;``` [L28](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultRegistry.sol#L28) (state variable declaration can not go after constructor)
1. ```mapping(bytes32 => mapping(bytes32 => Template)) public templates;``` [L31](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L31) (state variable can not go after constructor)
1. ```function convertToUnderlyingShares(uint256, uint256 shares)``` [L129](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L129) (public function can not go after internal function)
1. ```mapping(bytes32 => Escrow) public escrows;``` [L64](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L64) (state variable can not go after external function)
1. ```function deposit(uint256 _amount) external returns (uint256) {``` [L75](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L75) (external function can not go after public function)

**Description:**

According to [official solidity documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) functions should be grouped according to their visibility and ordered:

+ constructor

+ receive function (if exists)

+ fallback function (if exists)

+ external

+ public

+ internal

+ private

Within a grouping, place the view and pure functions last.

**Recommendation:**

Put the functions in the correct order according to the documentation.

### [N-4]: Typos
**Context:**

1. ```* Is used by `VaultController` to check if a target is a registerd clone.``` [L14](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L14) (Change **registerd** to **registered**)
1. ```* @param template Contains the implementation address and necessary informations to clone the implementation.``` [L65](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L65) (Change **informations** to **information**)
1. ```/// @notice The amount of assets that are free to be withdrawn from the yVault after locked profts.``` [L100](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L100) (Change **profts** to **profits**)
1. ```* @dev This function is the one stop solution to create a new vault with all necessary admin functions or auxiliery contracts.``` [L87](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L87) (Change **auxiliery** to **auxiliary**)
1. ```// Dont wait more than X seconds``` [L792](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L792) (Change **dont** to **don't**)
1. ```* @notice Sets a new `DeploymentController` and saves its auxilary contracts. Caller must be owner.``` [L829](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L829) (Change **auxilary** to **auxiliary**)
1. ```/// @Notice Optional - Metadata CID which can be used by the frontend to add informations to a vault/adapter...``` [L13](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/ITemplateRegistry.sol#L13) (Change **informations** to **information**)

### [N-5]: Line is too long
**Context:**

1. ```* AdminProxy is controlled by VaultController. VaultController executes management functions on other contracts through `execute()```` [L13](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L13) 
1. ```* @notice Registers a new vault with Metadata which can be used by a frontend. Caller must be owner. (`VaultController`)``` [L41](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultRegistry.sol#L41) 
1. ```* @notice Clones an implementation and initializes the clone. Caller must be owner.  (`VaultController` via `AdminProxy`)``` [L91](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L91) 
1. ```* @dev The basic templateCategories will be added via `VaultController` they are ("Vault", "Adapter", "Strategy" and "Staking").``` [L49](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L49) 
1. ```* @dev there is no check to ensure that all escrows are owned by the same account. Make sure to account for this either by only sending ids for a specific account or by filtering the Escrows by account later on.``` [L49](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L49) 
1. ```* Only the owner can add new tokens as rewards. Once added they cant be removed or changed. RewardsSpeed can only be adjusted if the rewardsSpeed is not 0.``` [L22](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L22) 
1. ```* @notice Changes rewards speed for a rewardToken. This works only for rewards that accrue over time. Caller must be owner.``` [L291](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L291) 
1. ```// If a deposit/withdraw operation gets called for another user we should accrue for both of them to avoid potential issues like in the Convex-Vulnerability``` [L380](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L380) 
1. ```* @dev Usually the adapter should already be pre configured. Otherwise a new one can only be added after a ragequit time.``` [L55](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L55) 
1. ```/// @return Maximum amount of underlying `asset` token that may be deposited for a given address. Delegates to adapter.``` [L398](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L398) 
1. ```/// @return Maximum amount of underlying `asset` token that can be withdrawn by `caller` address. Delegates to adapter.``` [L408](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L408) 
1. ```* @dev Management fee is annualized per minute, based on 525,600 minutes per year. Total assets are calculated using``` [L425](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L425) 
1. ```*  the average of their current value and the value at the previous fee harvest checkpoint. This method is similar to``` [L426](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L426) 
1. ```* @dev This function is the one stop solution to create a new vault with all necessary admin functions or auxiliery contracts.``` [L87](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L87) 
1. ```* @notice Changes rewards speed for a rewardToken. This works only for rewards that accrue over time. Caller must be creator of the Vault.``` [L477](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L477) 
1. ```/// @notice Pause Deposits and withdraw all funds from the underlying protocol. Caller must be owner or creator of the Vault.``` [L604](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L604) 
1. ```/// @notice Unpause Deposits and deposit all funds into the underlying protocol. Caller must be owner or creator of the Vault.``` [L630](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L630) 

**Description:**

Maximum suggested line length is [120 characters](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length).
