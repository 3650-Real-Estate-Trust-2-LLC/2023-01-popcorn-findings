# [G-01] ++i/i++ should be unchecked{ ++i } / unchecked{ i++ } when its not posssible for them to overflow, as its in the case when used in for loops/while loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

## Proof Of Concept
```solidity
for (uint256 i = 0; i < escrowIds.length; i++) {
```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L53

```solidity
for (uint256 i = 0; i < escrowIds.length; i++) {
```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L155

```solidity
for (uint256 i = 0; i < tokens.length; i++) {
```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L210

```solidity
for (uint256 i = 0; i < len; i++) {
```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L42

