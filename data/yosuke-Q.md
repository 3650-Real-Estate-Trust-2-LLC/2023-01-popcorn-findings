## [YO NC-1] Constants should be defined rather than using magic numbers

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L85
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L211

### Recommended Mitigation Steps
Constants should be defined rather than using magic numbers

## [YO NC-2] Unlocked Pragma

### Handle
yosuke

## Vulnerability details
### Impact
Every Solidity file specifies in the header a version number of the format pragma solidity ^0.8.0. The caret (^) before the version number implies an unlocked pragma, meaning that the compiler will use the specified version or above.

It’s usually a good idea to pin a specific version to know what compiler bug fixes and optimizations were enabled at the time of compiling the contract.

### Proof of Concept
all files

### Recommended Mitigation Steps
ex)
before

```solidity=
pragma solidity ^0.8.15;
```

after

```solidity=
pragma solidity 0.8.17;
```