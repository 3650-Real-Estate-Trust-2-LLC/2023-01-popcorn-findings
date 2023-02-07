# [G-01] Reorder structure layout

## Context
[IMultiRewardEscrow.sol#L7-L22](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/IMultiRewardEscrow.sol#L7-L22), [IMultiRewardStaking.sol#L14-L26](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/IMultiRewardStaking.sol#L14-L26), [IMultiRewardStaking.sol#L28-L35](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/IMultiRewardStaking.sol#L28-L35), [ITemplateRegistry.sol#L8-L21](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/ITemplateRegistry.sol#L8-L21), [IVaultRegistry.sol#L7-L22](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVaultRegistry.sol#L7-L22)

## Recommendations
The following structs could be optimized moving the position of certains values in order to save slot storages:   

[IMultiRewardEscrow.sol#L7-L22](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/IMultiRewardEscrow.sol#L7-L22)
````solidity
            struct Escrow {
              /// @notice The escrowed token
              IERC20 token;
+              /// @notice Owner of the escrow
+              address account;              
              /// @notice Timestamp of the start of the unlock
              uint32 start;
              /// @notice The timestamp the unlock ends at
              uint32 end;
              /// @notice The timestamp the index was last updated at
              uint32 lastUpdateTime;
              /// @notice Initial balance of the escrow
              uint256 initialBalance;
              /// @notice Current balance of the escrow
              uint256 balance;
-              /// @notice Owner of the escrow
-              address account;
            }
````
[IMultiRewardStaking.sol#L14-L26](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/IMultiRewardStaking.sol#L14-L26), [IMultiRewardStaking.sol#L28-L35](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/IMultiRewardStaking.sol#L28-L35)
````solidity
            struct RewardInfo {
+              /// @notice The timestamp the rewards end at
+              /// @dev use 0 to specify no end
+              uint32 rewardsEndTimestamp;
+              /// @notice The timestamp the index was last updated at
+              uint32 lastUpdatedTimestamp;
              /// @notice scalar for the rewardToken
              uint64 ONE;
              /// @notice Rewards per second
              uint160 rewardsPerSecond;
-              /// @notice The timestamp the rewards end at
-              /// @dev use 0 to specify no end
-              uint32 rewardsEndTimestamp;
              /// @notice The strategy's last updated index
              uint224 index;
-              /// @notice The timestamp the index was last updated at
-              uint32 lastUpdatedTimestamp;
            }

            struct EscrowInfo {
+              /// @notice Duration of the escrow in seconds
+              uint32 escrowDuration;
+              /// @notice A cliff before the escrow starts in seconds
+              uint32 offset;            
              /// @notice Percentage of reward that gets escrowed in 1e18 (1e18 = 100%, 1e14 = 1 BPS)
              uint192 escrowPercentage;
-              /// @notice Duration of the escrow in seconds
-              uint32 escrowDuration;
-              /// @notice A cliff before the escrow starts in seconds
-              uint32 offset;
            }
````
[ITemplateRegistry.sol#L8-L21](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/ITemplateRegistry.sol#L8-L21)
````solidity
            struct Template {
+              /// @Notice implementations can only be cloned if endorsed
+              bool endorsed;
+              /// @Notice If true, the implementation will require an init data to be passed to the clone function
+              bool requiresInitData;
+              /// @Notice Optional - Address of an registry which can be used in an adapter initialization
+              address registry;
              /// @Notice Cloneable implementation address
              address implementation;
-              /// @Notice implementations can only be cloned if endorsed
-              bool endorsed;
              /// @Notice Optional - Metadata CID which can be used by the frontend to add informations to a vault/adapter...
              string metadataCid;
-              /// @Notice If true, the implementation will require an init data to be passed to the clone function
-              bool requiresInitData;
-              /// @Notice Optional - Address of an registry which can be used in an adapter initialization
-              address registry;
              /// @Notice Optional - Only used by Strategies. EIP-165 Signatures of an adapter required by a strategy
              bytes4[8] requiredSigs;
            }
````
[IVaultRegistry.sol#L7-L22](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVaultRegistry.sol#L7-L22)
````solidity
            struct VaultMetadata {
              /// @notice Vault address
              address vault;
              /// @notice Staking contract for the vault
              address staking;
              /// @notice Owner and Vault creator
              address creator;
+              /// @notice OPTIONAL - If the asset is an Lp Token its the pool address
+              address swapAddress;              
              /// @notice IPFS CID of vault metadata
              string metadataCID;
              /// @notice OPTIONAL - If the asset is an Lp Token these are its underlying assets
              address[8] swapTokenAddresses;
-              /// @notice OPTIONAL - If the asset is an Lp Token its the pool address
-              address swapAddress;
              /// @notice OPTIONAL - If the asset is an Lp Token this is the identifier of the exchange (1 = curve)
              uint256 exchange;
            }
````


# [G-02] Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

## Context
[IMultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/IMultiRewardEscrow.sol), [IMultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/IMultiRewardStaking.sol), [IVault.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVault.sol), [IVaultController.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVaultController.sol), [MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol), [MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol), [Vault.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol), [VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol), [AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol)

## Description
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.  

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html  

Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.    
[IMultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/IMultiRewardEscrow.sol)
````solidity
11            uint32 start;
12            /// @notice The timestamp the unlock ends at
13            uint32 end;
14            /// @notice The timestamp the index was last updated at
15            uint32 lastUpdateTime;

36            uint32 duration,
37            uint32 offset
````
[IMultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/IMultiRewardStaking.sol)

````solidity
16            uint64 ONE; // @audit : gas - small uints
17            /// @notice Rewards per second
18            uint160 rewardsPerSecond; // @audit : gas - small uints
19            /// @notice The timestamp the rewards end at
20            /// @dev use 0 to specify no end
21            uint32 rewardsEndTimestamp; // @audit : gas - small uints
22            /// @notice The strategy's last updated index
23            uint224 index; // @audit : gas - small uints
24            /// @notice The timestamp the index was last updated at
25            uint32 lastUpdatedTimestamp; // @audit : gas - small uints

30            uint192 escrowPercentage; // @audit : gas - small uints
31            /// @notice Duration of the escrow in seconds
32            uint32 escrowDuration; // @audit : gas - small uints
33            /// @notice A cliff before the escrow starts in seconds
34            uint32 offset; // @audit : gas - small uints

40            uint160 rewardsPerSecond, // @audit : gas - small uints

43            uint192 escrowPercentage, // @audit : gas - small uints
44            uint32 escrowDuration, // @audit : gas - small uints
45            uint32 offset // @audit : gas - small uints

48            function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external;
````
[IVault.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVault.sol)
````solidity
9            uint64 deposit; // @audit : gas - small uints
10            uint64 withdrawal; // @audit : gas - small uints
11            uint64 management; // @audit : gas - small uints
12            uint64 performance; // @audit : gas - small uints
````
[IVaultController.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVaultController.sol)
````solidity
60            uint160[] memory rewardsSpeeds // @audit : gas - small uints
````
[MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol)
````solidity
73            event Locked(IERC20 indexed token, address indexed account, uint256 amount, uint32 duration, uint32 offset);

92            uint32 duration, // @audit : gas - small uints
93            uint32 offset // @audit : gas - small uints

114            uint32 start = block.timestamp.safeCastTo32() + offset;
````
[MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol)

````solidity
33            uint8 private _decimals;

67            function decimals() public view override(ERC20Upgradeable, IERC20Metadata) returns (uint8) {

220            event RewardInfoUpdate(IERC20 rewardToken, uint160 rewardsPerSecond, uint32 rewardsEndTimestamp);

245            uint160 rewardsPerSecond,

248            uint192 escrowPercentage, // @audit : gas - small uints
249            uint32 escrowDuration, // @audit : gas - small uints
250            uint32 offset // @audit : gas - small uints

274            uint64 ONE = (10**IERC20Metadata(address(rewardToken)).decimals()).safeCastTo64(); // @audit : gas - small uints
275            uint32 rewardsEndTimestamp = rewardsPerSecond == 0 // @audit : gas - small uints

296            function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {

307            uint32 prevEndTime = rewards.rewardsEndTimestamp; // @audit : gas - small uints
308            uint32 rewardsEndTimestamp = _calcRewardsEnd( // @audit : gas - small uints

340            uint32 rewardsEndTimestamp = rewards.rewardsEndTimestamp;

352            uint32 rewardsEndTimestamp, // @audit : gas - small uints
353            uint160 rewardsPerSecond, // @audit : gas - small uints

355            ) internal returns (uint32) {

373            for (uint8 i; i < _rewardTokens.length; i++) {

404            uint224 deltaIndex;

450            uint8 v,
````
[Vault.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol)
````solidity
38            uint8 internal _decimals;

100            function decimals() public view override returns (uint8) {

669            uint8 v,
````
[VaultController.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol)
````solidity
314            uint8 len = uint8(vaults.length);

319            for (uint8 i = 0; i < len; i++) {

336            uint8 len = uint8(vaults.length); // @audit : gas - small uints// @audit : gas - small uints
337            for (uint8 i = 0; i < len; i++) { // @audit : gas - small uints

353            uint8 len = uint8(vaults.length);

357            for (uint8 i = 0; i < len; i++) { 

373            uint8 len = uint8(vaults.length); // @audit : gas - small uints // @audit : gas - small uints
374            for (uint8 i = 0; i < len; i++) { // @audit : gas - small uints

436            uint8 len = uint8(vaults.length);

440            uint160 rewardsPerSecond,

443            uint224 escrowDuration, // @audit : gas - small uints
444            uint24 escrowPercentage, // @audit : gas - small uints

446            ) = abi.decode(rewardTokenData[i], (address, uint160, uint256, bool, uint224, uint24, uint256));

486            uint160[] calldata rewardsSpeeds

488            uint8 len = uint8(vaults.length);

517            uint8 len = uint8(vaults.length);

563            uint8 len = uint8(templateCategories.length);

583            len = uint8(templateCategories.length);

606            len = uint8(vaults.length);

619            uint8 len = uint8(vaults.length);

632            uint8 len = uint8(vaults.length);

645            uint8 len = uint8(vaults.length);

765            uint8 len = uint8(adapters.length);

805            uint8 len = uint8(adapters.length);
````
[AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol)
````solidity
38            uint8 internal _decimals;

93            returns (uint8)

637            uint8 v,
````


# [G-03] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a `struct`, where appropriate

## Context
[MultiRewardEscrow.sol#L67-L69](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L67-L69), [MultiRewardStaking.sol#L211-L213](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L211-L213), [MultiRewardStaking.sol#L216-L218](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L216-L218), [MultiRewardStaking.sol#L440](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L440), [TemplateRegistry.sol#L31-L35](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L31-L35), [VaultRegistry.sol#L28-L31](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L28-L31)

## Description
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.     
[MultiRewardEscrow.sol#L67-L69](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L67-L69)
````solidity
            mapping(address => bytes32[]) public userEscrowIds;
            // User => RewardsToken => Escrows
            mapping(address => mapping(IERC20 => bytes32[])) public userEscrowIdsByToken;
````
[MultiRewardStaking.sol#L211-L213](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L211-L213), [MultiRewardStaking.sol#L216-L218](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L216-L218), [MultiRewardStaking.sol#L440](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L440)
````solidity
            mapping(IERC20 => RewardInfo) public rewardInfos;
            // rewardToken -> EscrowInfo
            mapping(IERC20 => EscrowInfo) public escrowInfos;
            
            
            mapping(address => mapping(IERC20 => uint256)) public userIndex;
            // user => rewardToken -> accruedRewards
            mapping(address => mapping(IERC20 => uint256)) public accruedRewards;

            mapping(address => uint256) public nonces;
````
[TemplateRegistry.sol#L31-L35](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L31-L35)
````solidity
            mapping(bytes32 => mapping(bytes32 => Template)) public templates;
            mapping(bytes32 => bytes32[]) public templateIds;
            mapping(bytes32 => bool) public templateExists;
            mapping(bytes32 => bool) public templateCategoryExists;
````
[VaultRegistry.sol#L28-L31](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L28-L31)
````solidity
            mapping(address => VaultMetadata) public metadata;
            // asset to vault addresses
            mapping(address => address[]) public vaultsByAsset;
````
# [G-04] `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables
## Recommendations
Consider implementing this formula instead.            
# [G-05] `<x> > <y>` Costs More Gas Than `<x> >= <y>`            
## Recommendations
Consider implementing `>=` instead.