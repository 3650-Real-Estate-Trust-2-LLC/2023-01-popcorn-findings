### 1. Lock the Pragma or decrease it.
Contracts should be deployed with the same compiler version, that they have been tested thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, a compiler version that might introduce bugs that affect the contract system negatively. - [SWC-103](https://swcregistry.io/docs/SWC-103).

Also, if you want floating pragma so other projects can integrate Drips contracts, then must consider dropping the pragma. ^0.8.15 is a recent version.  I would recommend using at least 0.8.13 in that case. As most projects use 0.8.7-0.8.13 solidity compiler versions, which are considered more stable compiler versions. Or Simply lock the pragma, `0.8.15`.

### 2. Loss of precision due to rounding.
```solidity
        if (lockedFundsRatio < DEGRADATION_COEFFICIENT) {
            uint256 lockedProfit = yVault.lockedProfit();
            return lockedProfit - ((lockedFundsRatio * lockedProfit) / DEGRADATION_COEFFICIENT);  /// @audit-low loss of precision
        } else {
            return 0;
        }
```
Add scalars so roundings are negligible.

These are the instances of this issue:
1. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L93
2. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L122
3. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L359
4. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L343
5. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L433
6. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L454
7. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L536

### 3. Wrong Function Ordering.
Project's contracts don't follow the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#), especially in function ordering. Many `view` functions were defined before public/external/internal functions & some private functions were defined in between public/external functions. It was confusing & was making the audit process a little difficult. Pls consider using this order in your contracts for better readability:
```
constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
Within a grouping, place the view and pure functions last.
```
The wrong function ordering is in every contract.

### 4. Capital letters should be used only for constant values not other state variables or function names etc.
It increases the code readability and understanding. 

These are the instances of this issue:
1. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L709
2. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L517
3. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L74
4. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L75
5. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L438
6. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L439
7. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L657
8. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L658
9. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L36
10. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L37
11. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L38
12. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L39
13. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L40

### 5. These `Immutable` variables should be defined as `constant`.
1. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L36
2. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L37
3. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L38
4. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L39
5. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L40

### 6. Solidity's layout was not followed.
Almost None of the follows [solidity layout](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#).
This [mapping](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L659) is defined after literally 600+ lines of code. 
It is pretty much clear that they used their own layout but still the best-recommended practice should be used as defined in solidity docs. These little things make it a bit harder to understand the code. 

### 7. Shadowed state variable name
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L62