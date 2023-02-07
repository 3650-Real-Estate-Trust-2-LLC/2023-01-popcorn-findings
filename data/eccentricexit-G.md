# Save 2 SLOADs in Vault.accruedPerformanceFee.

The code saves `highWaterMark` to memory but then reads from storage again. Save 2 SLOADs by reading from memory (with `highWaterMark_`) in line [L453](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L453) and [L455](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L455):

```diff
    function accruedPerformanceFee() public view returns (uint256) {
        uint256 highWaterMark_ = highWaterMark;
        uint256 shareValue = convertToAssets(1e18);
        uint256 performanceFee = fees.performance;

        return
-            performanceFee > 0 && shareValue > highWaterMark
+            performanceFee > 0 && shareValue > highWaterMark_
                ? performanceFee.mulDiv(
-                    (shareValue - highWaterMark) * totalSupply(),
+                    (shareValue - highWaterMark_) * totalSupply(),
                    1e36,
                    Math.Rounding.Down
                )
                : 0;
    }
```