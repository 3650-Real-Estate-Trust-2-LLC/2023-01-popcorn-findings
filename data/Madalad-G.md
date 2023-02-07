# Gas Report

## Use assembly to check for `address(0)`

Saves 16000 deployment gas per instance and 6 runtime gas per instance.

```
assembly {
 if iszero(_addr) {
  mstore(0x00, "zero address")
  revert(0x00, 0x20)
 }
}
```

Instances: 25
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L45
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L95
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L96
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L62
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L77
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L82
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L138
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L198
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L209
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L222
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L142
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L481
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L74
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L90
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L141
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L177
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L216
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L258
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L554
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L702
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L108
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L682
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L687
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L693
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L837

<br>

## Use assembly to calculate hashes

Saves 5000 deployment gas per instance and 80 runtime gas per instance.

### Unoptimized
```
function solidityHash(uint256 a, uint256 b) public view {
	//unoptimized
	keccak256(abi.encodePacked(a, b));
}
```

### Optimized
```
function assemblyHash(uint256 a, uint256 b) public view {
	//optimized
	assembly {
		mstore(0x00, a)
		mstore(0x20, b)
		let hashedVal := keccak256(0x00, 0x40)
	}
}
```

Instances: 17
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L104
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L460
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L464
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L466
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L493
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L495
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L496
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L497
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L93
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L679
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L683
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L685
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L718
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L720
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L723
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L724
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L40

<br>

## Cache storage variables rather than re-reading from storage

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.

Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata.

Instances: 2
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L448
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L453

<br>

## Use `indexed` to save gas
Using `indexed` for value type event parameters (address, uint, bool) saves at least 80 gas in emitting the event (https://gist.github.com/Tomosuke0930/9bf61e01a8c3e214d95a9b84dcb41d97)

Note that for other types however (string, bytes), it is more expensive

Instances: 31
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L28
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L28
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L28
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneRegistry.sol#L33
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L36
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L39
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L91
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L92
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L73
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L73
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L73
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L136
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L196
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L159
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L159
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L220
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L111
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L112
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L118
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L119
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L512
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L514
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L514
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L569
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L621
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L741
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L741
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L781
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L781
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L824
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L824

<br>

## Use `constant`, `immutable` where necessary

Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD. Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas)

Instances: 9
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L23
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L24
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L25
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L191
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L467
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L387
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L535
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L717
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L822

<br>

## Use `private` rather than `public` for constants

Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table. If needed to be viewed externally, the values can be read from the verified contract source code.

Instances: 1
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L31

<br>

## `x += y` costs more gas than `x = x + y` for state variables

Instances: 7
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L408
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L431

<br>

## Usage of `uint` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

Consider using a larger size then downcasting where needed.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

Instances: 2
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L33
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L38

<br>
