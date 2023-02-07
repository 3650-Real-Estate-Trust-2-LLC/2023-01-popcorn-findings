## [NAZ-G1] Unused Events and Errors
**Severity**: Gas
**Context**: [`TemplateRegistry.sol#L40`](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L40), [`MultiRewardEscrow.sol#L200`](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L200), [`MultiRewardStaking.sol#L226`](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L226)

**Description**:
There are a few events and errors that are not used across several contracts. These unused events and errors can be removed to save gas.

**Recommendation**:
Consider removing them to save gas or using them.