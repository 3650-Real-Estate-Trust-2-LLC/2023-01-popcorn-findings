# Stacking the Storage Values in Ascending Order (Smallest - Biggest)

Through placing the storage values found in the `lock` function, a decrease in gas fees should be seen as without it, shifts within the contract need to be made adding one more action to the contract, which should result in an increase in gas

File: `src/utils/MultiRewardEscrow.sol`

Line 88 - 93
## Before - 
` uint256 amount,
uint32 duration,
uint32 offset`

## After - 
`unit32 duration, unit32 offset, unit256 amount`

As well as:

File: `src/utils/MultiRewardStaking.sol`
Line 248-250
## Before - 
    `uint192 escrowPercentage,
    uint32 escrowDuration,
    uint32 offset`
## After:
    `uint32 escrowDuration,
    uint32 offset,
    uint192 escrowPercentage`
 
