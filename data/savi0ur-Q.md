Anyone able to call `changeFees` function from Vault multiple times after `block.timestamp < proposedFeeTime + quitPeriod` satisfied, consider updating proposedFeeTime = 0

```solidity
function changeFees() external {
    if (block.timestamp < proposedFeeTime + quitPeriod)
        revert NotPassedQuitPeriod(quitPeriod);

    emit ChangedFees(fees, proposedFees);
    fees = proposedFees;
}
```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L540-L546