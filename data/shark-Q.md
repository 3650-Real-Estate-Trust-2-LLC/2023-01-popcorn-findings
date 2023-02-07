## 1. Order of functions

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.

Functions should be grouped according to their visibility and ordered:

- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private

Source: [docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions)

1. [src/vault/adapter/beefy/BeefyAdapter.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol) (public functions should be before internal functions)

## 2. Consider declaring constants instead of magic numbers

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L630

```solidity
        if (_quitPeriod < 1 days || _quitPeriod > 7 days)
```

Instead of using magic numbers `1 days` and `7 days`, consider creating `constant` variables i.e. `MIN_QUIT_PERIOD`, `MAX_QUIT_PERIOD`.

## 3. Unspecific Compiler Version Pragma

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

For example, the version that is currently used (`pragma solidity ^0.8.15;`) is unspecific.

## 4. Use a more recent version of solidity

The current version being used is `0.8.15`. However, it is considered best practice to use the latest version of solidity (currently `0.8.17`). More recent versions of solidity have compiler optimizations among other things. This could help in reading and writing safe and clean code.
