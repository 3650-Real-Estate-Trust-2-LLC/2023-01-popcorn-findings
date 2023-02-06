
## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | State variables only set in the constructor should be declared `immutable` | 8 |  106887 |
| [G&#x2011;02] | Using `storage` instead of `memory` for structs/arrays saves gas | 2 |  4200 |
| [G&#x2011;03] | State variables can be packed into fewer storage slots | 1 |  20000 |
| [G&#x2011;04] | `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops | 22 |  1290 |
| [G&#x2011;05] | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables (`-=` too) | 5 |  565 |

Total: 38 instances over 5 issues with **132,942 gas** saved.

Gas totals use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions.

## Gas Optimizations

### [G&#x2011;01]  State variables only set in the constructor should be declared `immutable`
Avoids a Gsset (**20000 gas**) in the constructor, and replaces the first access in each transaction (Gcoldsload - **2100 gas**) and each access thereafter (Gwarmaccess - **100 gas**) with a `PUSH32` (**3 gas**). 

While `string`s are not value types, and therefore cannot be `immutable`/`constant` if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract `abstract` with `virtual` functions for the `string` accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

*There are 8 instances of this issue:*

```solidity
File: src\utils\MultiRewardEscrow.sol

/// @audit 1 Gcoldsloads, 0 Gwarmsloads
191   address public feeRecipient;
```

```solidity
File: src\vault\DeploymentController.sol

/// @audit 3 Gcoldsloads, 0 Gwarmsloads
23    ICloneFactory public cloneFactory;

/// @audit 3 Gcoldsloads, 0 Gwarmsloads
24    ICloneRegistry public cloneRegistry;

/// @audit 6 Gcoldsloads, 0 Gwarmsloads
25    ITemplateRegistry public templateRegistry;
```

```solidity
File: src\vault\VaultController.sol

/// @audit 4 Gcoldsloads, 1 Gwarmsloads
387   IVaultRegistry public vaultRegistry;

/// @audit 2 Gcoldsloads, 0 Gwarmsloads
535   IMultiRewardEscrow public escrow;

/// @audit 24 Gcoldsloads, 18 Gwarmsloads
717   IAdminProxy public adminProxy;

/// @audit 3 Gcoldsloads, 2 Gwarmsloads
822   IPermissionRegistry public permissionRegistry;
```

### [G&#x2011;02]  Using `storage` instead of `memory` for structs/arrays saves gas
When fetching data from a storage location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (**2100 gas**) for *each* field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declaring the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct.

*There are 2 instances of this issue:*

```solidity
File: src\utils\MultiRewardEscrow.sol

/// @audit initialBalance (1 Gcoldsload)
157       Escrow memory escrow = escrows[escrowId];
```

```solidity
File: src\utils\MultiRewardStaking.sol

/// @audit ONE, rewardsPerSecond, rewardsEndTimestamp (1 Gcoldsload)
254     RewardInfo memory rewards = rewardInfos[rewardToken];
```

### [G&#x2011;03]  State variables can be packed into fewer storage slots
If variables occupying the same slot are both written by the same function or by the constructor, avoids a separate Gsset (**20000 gas**). Reads of the variables can also be cheaper.

*There is 1 instance of this issue:*

```solidity
File: src\vault\adapter\abstracts\AdapterBase.sol

/// @audit _decimals + strategy (1 Gsset)
38      uint8 internal _decimals;
427     IStrategy public strategy;
```

### [G&#x2011;04]  `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops
The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**.

*There are 22 instances of this issue:*

```solidity
File: src\utils\MultiRewardEscrow.sol

53      for (uint256 i = 0; i < escrowIds.length; i++) {

102     nonce++;

155     for (uint256 i = 0; i < escrowIds.length; i++) {

210     for (uint256 i = 0; i < tokens.length; i++) {
```

```solidity
File: src\utils\MultiRewardStaking.sol

171     for (uint8 i; i < _rewardTokens.length; i++) {

373     for (uint8 i; i < _rewardTokens.length; i++) {
```

```solidity
File: src\vault\PermissionRegistry.sol

42      for (uint256 i = 0; i < len; i++) {
```

```solidity
File: src\vault\VaultController.sol

319     for (uint8 i = 0; i < len; i++) {

337     for (uint8 i = 0; i < len; i++) {

357     for (uint8 i = 0; i < len; i++) {

374     for (uint8 i = 0; i < len; i++) {

437     for (uint256 i = 0; i < len; i++) {

494     for (uint256 i = 0; i < len; i++) {

523     for (uint256 i = 0; i < len; i++) {

564     for (uint256 i = 0; i < len; i++) {

587     for (uint256 i = 0; i < len; i++) {

607     for (uint256 i = 0; i < len; i++) {

620     for (uint256 i = 0; i < len; i++) {

633     for (uint256 i = 0; i < len; i++) {

646     for (uint256 i = 0; i < len; i++) {

766     for (uint256 i = 0; i < len; i++) {

806     for (uint256 i = 0; i < len; i++) {
```

### [G&#x2011;05]  `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables (`-=` too)
Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**. Subtructions act the same way.

*There are 5 instances of this issue:*

```solidity
File: src\utils\MultiRewardStaking.sol

408     rewardInfos[_rewardToken].index += deltaIndex;

431     accruedRewards[_user][_rewardToken] += supplierDelta;
```

```solidity
File: src\vault\adapter\abstracts\AdapterBase.sol

158         underlyingBalance += _underlyingBalance() - underlyingBalance_;

225             underlyingBalance -= underlyingBalance_ - _underlyingBalance();
```

```solidity
File: src\utils\MultiRewardEscrow.sol

162       escrows[escrowId].balance -= claimable;
```
