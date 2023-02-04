https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneRegistry.sol#L41

In the function "addClone" when you try to add an already added clone it will let you pass through,
So I suggest that you add a check to validate if a clone is already present,
Below is code for the same

// code
// check if cloneExists[clone] is false then only add a new clone
require(cloneExists[clone] == false, "Already added clone");