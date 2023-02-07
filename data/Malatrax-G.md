## AccruedPerformanceFee waste of gas Storage variable read multiple times
### Details
[accruedPerformanceFee source code GitHub Link](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L441-L460)

The local variable `highWaterMark_` mirroring the storage variable `highWaterMark` isn't used, the storage variable is instead used. Which means that the storage is read multiple times and the local variable unused, leading to a waste of gas

### Recommended Mitigation Steps
Use the defined local variable `highWaterMark_` instead of the storage in the `accruedPerformanceFee`function.