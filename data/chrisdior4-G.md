## Gas Optimization

| G-O    |Issue|
|:------:|:----|
| [G&#x2011;01] | Use of _msgSender() when there is no implementation of EIP-2771 | 2 |
| [G&#x2011;02] |  x = x - y costs less gas than x -= y, same for addition | 6 |
| [G&#x2011;03] | Some public functions can be changed to external | 11 |
| [G&#x2011;04] | Use calldata instead of memory for arguments in external functions | 19 |
| [G&#x2011;05] | Use x != 0 instead of x > 0 for uint types | 31 |
| [G&#x2011;06] | ++i costs 5 gas less than i++ | 2 |
| [G&#x2011;07] | ++I/I++ should be UNCHECKED{++I}/UNCHECKED{I++}  when it is not possible for them to overflow | 1 |
| [G&#x2011;08] | Using STORAGE instead of MEMORY for structs/arrays saves gas -3 | 1 |

Total: 8 issues


### [G-01] Use of _msgSender() when there is no implementation of EIP-2771 

Replace _msgSender() with msg.sender if there is no mechanism to support meta-transactions like EIP-2771 implemented.

File: `AdapterBase.sol`

There are 4 occurances here:

```solidity
_deposit(_msgSender(), receiver, assets, shares);
```
```solidity
 _deposit(_msgSender(), receiver, assets, shares);
```
```solidity
 _withdraw(_msgSender(), receiver, owner, assets, shares);
 ```
 ```solidity
_withdraw(_msgSender(), receiver, owner, assets, shares);
```

Lines of code:

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L119

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L138

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L182

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L201


### [G-02] x = x - y costs less gas than x -= y, same for addition

You can replace all -= and += occurrences to save gas

Example in `AdapterBase.sol`:

```solidity
underlyingBalance += _underlyingBalance() - underlyingBalance_;
```

Lines of code:

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L158


### [G-03] Some public functions can be changed to external

Since a function is not called from within a contract, using external instead of public will save gas as all the function arguments will be in calldata instead of copied to memory

File: `AdapterBase.sol`

```solidity
function previewRedeem(uint256 shares)
        public
        view
        virtual
        override
        returns (uint256)
    {
```
```solidity
function strategyDeposit(uint256 amount, uint256 shares)
        public
        onlyStrategy
    {
```
```solidity
function strategyWithdraw(uint256 amount, uint256 shares)
        public
        onlyStrategy
    {
```

Lines of code:

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L353

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L456

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L467

	
### [G-04] Use calldata instead of memory for arguments in external functions

By marking the function as external it is possible to use calldata in the arguments shown below and save significant gas.

File: `VaultController.sol`

```solidity
function deployVault(
    VaultInitParams memory vaultData,
    DeploymentArgs memory adapterData,
    DeploymentArgs memory strategyData,
    address staking,
    bytes memory rewardsData,
    VaultMetadata memory metadata,
    uint256 initialDeposit
  ) external canCreate returns (address vault) {
```

Lines of code:

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L90

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L91

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L92

### [G-05] Use x != 0 instead of x > 0 for uint types

The != operator costs less gas than > and for uint types you can use it to check for non-zero values to save gas

File: `VaultController.sol`

```solidity
if (adapterData.id > 0)
```

```solidity
if (rewardsData.length > 0) _handleVaultStakingRewards(vault, rewardsData);
```

```solidity
if (initialDeposit > 0) {
```

```solidity
if (strategyData.id > 0) {
```

```solidity
if (adapterId > 0) revert AdapterConfigFaulty();
```

Lines of code:

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L103

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L112

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L169

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L211

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L694

### [G-06] ++i costs 5 gas less than i++

File: `VaultController.sol`

There are 15 occurances of this problem in this smart contract, here is an example:

```solidity
for (uint8 i = 0; i < len; i++) {
```

Lines of code:

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L319


### [G-07] ++I/I++ should be UNCHECKED{++I}/UNCHECKED{I++}  when it is not possible for them to overflow

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

File: `VaultController.sol`

There are 15 occurances of this problem in this smart contract, here is an example:

```solidity
for (uint8 i = 0; i < len; i++) {
```

Lines of code:

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L319

All the occurances in this contract cannot overflow, so you can rewrite the `i++` to be unchecked.


### [G-08] Using STORAGE instead of MEMORY for structs/arrays saves gas -3

 When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct.

File: `MultiRewardStaking.sol`

```solidity
EscrowInfo memory escrowInfo = escrowInfos[_rewardTokens[i]];
```

```solidity
RewardInfo memory rewards = rewardInfos[rewardToken];
```

Lines of code:

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L176

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L254


