# Gas Optimizations

## Summary

|               | Issue         | Instances     |
| :-------------: |:-------------|:-------------:|
| 1  | State variables only set in the constructor should be declared `immutable`  | 8 |
| 2  | Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct | 9 |
| 3  | Variables inside struct should be packed to save gas  | 1  |
| 4  | Using `storage` instead of `memory` for struct/array saves gas | 2 |
| 5  | Use `unchecked` blocks to save gas  |  3 |
| 6  | `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables  |  5 |
| 7  | Duplicated input check statements should be refactored to a modifier  |  4 |
| 8  | Input check statements should be placed at the start of the functions |  3 |
| 9  | `memory` values should be emitted in events instead of `storage` ones  |  1 |
| 10 | `++i/i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as in the case when used in for & while loops |  21  |


## Findings


### 1- State variables only set in the constructor should be declared `immutable` :

Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmacces (100 gas) with a PUSH32 (3 gas).

There are 8 instances of this issue:

File: vault/DeploymentController.sol [Line 23-25](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L23-L25)
```
ICloneFactory public cloneFactory;
ICloneRegistry public cloneRegistry;
ITemplateRegistry public templateRegistry;
```

File: utils/MultiRewardEscrow.sol [Line 191](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L191)
```
address public feeRecipient;
```

File: vault/VaultController.sol [Line 387](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L387)
```
IVaultRegistry public vaultRegistry;
```

File: vault/VaultController.sol [Line 535](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L535)
```
IMultiRewardEscrow public escrow;
```

File: vault/VaultController.sol [Line 717](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L717)
```
IAdminProxy public adminProxy;
```

File: vault/VaultController.sol [Line 822](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L822)
```
IPermissionRegistry public permissionRegistry;
```

### 2- Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct :

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

There are 3 instances of this issue in `TemplateRegistry` :

```
File: vault/TemplateRegistry.sol

31      mapping(bytes32 => mapping(bytes32 => Template)) public templates;
32      mapping(bytes32 => bytes32[]) public templateIds;
35      mapping(bytes32 => bool) public templateCategoryExists;
```

These mappings could be refactored into the following struct and mapping for example :

```
struct TemplateCategory {
    bool templateCategoryExists;
    bytes32[] templateIds;
    mapping(bytes32 => Template) templates;
}
    
mapping(bytes32 => TemplateCategory) public templateCategoryMap;
```

There are 2 instances of this issue in `MultiRewardEscrow` :

```
File: utils/MultiRewardEscrow.sol

67      mapping(address => bytes32[]) public userEscrowIds;
69      mapping(address => mapping(IERC20 => bytes32[])) public userEscrowIdsByToken;
```

These mappings could be refactored into the following struct and mapping for example :

```
struct UserEscrow {
    bytes32[] userEscrowIds;
    mapping(IERC20 => bytes32[]) userEscrowIdsByToken;
}
    
mapping(address => UserEscrow) public userEscrows;
```

There are 4 instances of this issue in `MultiRewardStaking` :

```
File: utils/MultiRewardStaking.sol

211     mapping(IERC20 => RewardInfo) public rewardInfos;
213     mapping(IERC20 => EscrowInfo) public escrowInfos;
```

These mappings could be refactored into the following struct and mapping for example :

```
struct TokenInfo {
    RewardInfo rewardInfos;
    EscrowInfo escrowInfos;
}
    
mapping(IERC20 => TokenInfo) public infos;
```

The others instances are :

```
File: utils/MultiRewardStaking.sol

216     mapping(address => mapping(IERC20 => uint256)) public userIndex;
218     mapping(address => mapping(IERC20 => uint256)) public accruedRewards;
```

These mappings could be refactored into the following struct and mapping for example :

```
struct UserInfo {
    uint256 userIndex;
    uint256 accruedRewards;
}
    
mapping(address => mapping(IERC20 => UserInfo)) public userIndex;
```

### 3- Variables inside `struct` should be packed to save gas :

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

There is 1 instance of this issue:

File: interfaces/vault/IVaultRegistry.sol [Line 7-22](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVaultRegistry.sol#L7-L22)

```
struct VaultMetadata {
  /// @notice Vault address
  address vault;
  /// @notice Staking contract for the vault
  address staking;
  /// @notice Owner and Vault creator
  address creator;
  /// @notice IPFS CID of vault metadata
  string metadataCID;
  /// @notice OPTIONAL - If the asset is an Lp Token these are its underlying assets
  address[8] swapTokenAddresses;
  /// @notice OPTIONAL - If the asset is an Lp Token its the pool address
  address swapAddress;
  /// @notice OPTIONAL - If the asset is an Lp Token this is the identifier of the exchange (1 = curve)
  uint256 exchange;
}
```

As the `exchange` variable represents an id for different exchanges it can't really overflow **2^96**, so its value should be packed with the variable `swapAddress` (which only takes 20 bytes) in order to save gas, the struct should be modified as follow :

```
struct VaultMetadata {
  /// @notice Vault address
  address vault;
  /// @notice Staking contract for the vault
  address staking;
  /// @notice Owner and Vault creator
  address creator;
  /// @notice IPFS CID of vault metadata
  string metadataCID;
  /// @notice OPTIONAL - If the asset is an Lp Token these are its underlying assets
  address[8] swapTokenAddresses;
  /// @notice OPTIONAL - If the asset is an Lp Token its the pool address
  address swapAddress;
  /// @notice OPTIONAL - If the asset is an Lp Token this is the identifier of the exchange (1 = curve)
  uint96 exchange;
}
```

### 4- Using `storage` instead of `memory` for struct/array saves gas :

When fetching data from a `storage` location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from `storage`, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new `memory` variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. 

The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct.

There are 2 instances of this issue :

File: utils/MultiRewardStaking.sol [Line 372](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L372)
```
IERC20[] memory _rewardTokens = rewardTokens;
```

File: utils/MultiRewardStaking.sol [Line 414](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L414)
```
RewardInfo memory rewards = rewardInfos[rewardToken];
```

### 5- Use `unchecked` blocks to save gas :

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

There are 3 instances of this issue:

File: utils/MultiRewardStaking.sol [Line 392-396](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L392-L396)
```
if (rewards.rewardsEndTimestamp > block.timestamp) {
    elapsed = block.timestamp - rewards.lastUpdatedTimestamp;
} else if (rewards.rewardsEndTimestamp > rewards.lastUpdatedTimestamp) {
    elapsed = rewards.rewardsEndTimestamp - rewards.lastUpdatedTimestamp;
}
```

In code linked above both operation cannot underflow due to the checks that preceeds them so they both should be marked as `unchecked`. 

File: vault/Vault.sol [Line 147](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L147)
```
shares = convertToShares(assets) - feeShares;
```

In code linked above the operation cannot underflow because the calculated fee value is always less than  the share amount, so the operation should be marked as `unchecked`. 

File: utils/MultiRewardEscrow.sol [Line 110](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L110)
```
amount -= fee;
```

In code linked above the operation cannot underflow because the calculated fee value is always less than  the value of `amount`, so the operation should be marked as `unchecked`. 

### 6- `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables :

Using the addition operator instead of plus-equals saves **113 gas** as explained [here](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

There are 5 instances of this issue:

```
File: vault/PermissionRegistry.sol

158     underlyingBalance += _underlyingBalance() - underlyingBalance_;
225     underlyingBalance -= underlyingBalance_ - _underlyingBalance();

File: utils/MultiRewardEscrow.sol

162     escrows[escrowId].balance -= claimable;

File: utils/MultiRewardStaking.sol

408     rewardInfos[_rewardToken].index += deltaIndex;
431     accruedRewards[_user][_rewardToken] += supplierDelta;
```

### 7- Duplicated input check statements should be refactored to a modifier :

Input check statements used multiple times inside a contract should be refactored to a modifier for better readability and also to save gas.

There are 4 instances of this issue :

```
File: vault/Vault.sol

141     if (receiver == address(0)) revert InvalidReceiver();
177     if (receiver == address(0)) revert InvalidReceiver();
216     if (receiver == address(0)) revert InvalidReceiver();
258     if (receiver == address(0)) revert InvalidReceiver();
```

Those checks should be replaced by a `ValidReceiver` modifier as follow :

```
modifier ValidReceiver(address receiver){
    if (receiver == address(0)) revert InvalidReceiver();
    _;
}
```

### 8- Input check statements should be placed at the start of the functions :

The check statements on the functions input values should be placed at the beginning of the functions to avoid too deep stack errors and save gas.

There are 3 instances of this issue:

File: vault/Vault.sol [Line 74-75](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L74-L75)
```
if (address(asset_) == address(0)) revert InvalidAsset();
if (address(asset_) != adapter_.asset()) revert InvalidAdapter();
```

File: vault/Vault.sol [Line 90](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L90)
```
if (feeRecipient_ == address(0)) revert InvalidFeeRecipient();
```

### 9- `memory` values should be emitted in events instead of `storage` ones :

The values emitted in events shouldn’t be read from storage but the existing memory values should be used instead, this will save **~101 GAS**.

There is 1 instance of this issue :

File: vault/Vault.sol [Line 635](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L635)
```
emit QuitPeriodSet(quitPeriod);
```

The memory values should be emitted as follow :

```
emit QuitPeriodSet(_quitPeriod);
```

### 10- `++i/i++` should be `unchecked{++i}/unchecked{i++}` when it is not possible for them to overflow, as in the case when used in for & while loops :

There are 21 instances of this issue:
 
```
File: vault/PermissionRegistry.sol

42      for (uint256 i = 0; i < len; i++)

File: vault/VaultController.sol

318     for (uint8 i = 0; i < len; i++)
337     for (uint8 i = 0; i < len; i++)
357     for (uint8 i = 0; i < len; i++)
374     for (uint8 i = 0; i < len; i++)
437     for (uint256 i = 0; i < len; i++) 
494     for (uint256 i = 0; i < len; i++)
523     for (uint256 i = 0; i < len; i++)
564     for (uint256 i = 0; i < len; i++)
587     for (uint256 i = 0; i < len; i++)
607     for (uint256 i = 0; i < len; i++)
620     for (uint256 i = 0; i < len; i++)
633     for (uint256 i = 0; i < len; i++)
646     for (uint256 i = 0; i < len; i++)
766     for (uint256 i = 0; i < len; i++)
806     for (uint256 i = 0; i < len; i++)

File: utils/MultiRewardEscrow.sol

53      for (uint256 i = 0; i < escrowIds.length; i++)
155     for (uint256 i = 0; i < escrowIds.length; i++)
210     for (uint256 i = 0; i < tokens.length; i++)

File; utils/MultiRewardStaking.sol

171     for (uint8 i; i < _rewardTokens.length; i++)
373     for (uint8 i; i < _rewardTokens.length; i++)
```