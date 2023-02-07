## 1. [ L ] Use safeTransferFrom instead of transferFrom

**Description**:

Dev should use `safeTransferFrom` instead of `transferFrom` as best practice

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L526

**Recommendation**:
Use `safeTransferFrom`

```solidity
      rewardTokens[i].transferFrom(msg.sender, address(this), amounts[i]);
```

## 2. [ L ] Lock Pragmas to specific compiler version

**Description**:

One of the best practices for securing smart contracts is to lock the pragma version so that it is not accidentally deployed on a more recent version that has high risk of having undiscovered bugs.

Instances: All contracts

**Recommendation**:
Lock the pragmas to a specific compiler version
https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/

## 3. [ N ] Use bytes.concat() instead of abi.encodePacked()

**Description**:
Rather than using `abi.encodePacked()` for appending bytes, since version 0.8.4, bytes.concat() is enabled.

**References**:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L104
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L461
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L94
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L680
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L648

Total of 5 instances

**Recommendation:**
Use `bytes.concat()` instead.

## 4. [ N ]  Pragma version 0.8.15 is to recent to be trusted

**Description:**
https://github.com/ethereum/solidity/blob/develop/Changelog.md
0.8.15 (2022-06-15)
0.8.10 (2021-11-09)
Recent versions of pragma carry risk of having high severity bugs that have not yet been discovered and this can affect protocol in the future.

Instances: Used in all contracts.

**Recommendation:**
Use a legacy version that has been tested through time like 0.8.10