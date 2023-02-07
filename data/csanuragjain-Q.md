## Owner can accidentally add duplicate Clone

Contract:
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L41

Issue:
On adding a duplicate entry via `addClone` function, multiple clone will be written on `clones` and `allClones` object.

Impact:
If any future function relies directly on number of clones, or iterating allClones then it will encounter duplicate entry. if its not operated correctly then could result in incorrect outcome

Recommendation:
Add below condition:

```solidity
require(!cloneExists[clone], "Already exists");
```
