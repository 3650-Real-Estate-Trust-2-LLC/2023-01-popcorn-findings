# [N-01] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED()

Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled

Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,).

```solidity
104: bytes32 id = keccak256(abi.encodePacked(token, account, amount, nonce));
 ```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol

```solidity
49: _name = string(abi.encodePacked("Staked ", IERC20Metadata(address(_stakingToken)).name()));
50: _symbol = string(abi.encodePacked("pst-", IERC20Metadata(address(_stakingToken)).symbol()));
```

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#LL49C6-L50C7

```solidity
461: abi.encodePacked(
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L461

```solidity
648: abi.encodePacked(
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#LL648C20-L648C38
```solidity
94: abi.encodePacked("Popcorn", name(), block.timestamp, "Vault")
680:  abi.encodePacked(
```
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L94
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#LL680C20-L680C38