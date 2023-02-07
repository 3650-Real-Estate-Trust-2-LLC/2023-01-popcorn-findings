## Use of named returns for local variables saves gas
You can have further advantages in term of gas cost by simply using named return values as temporary local variable.

For instance, the code block below may be refactored as follows:

[File: WithRewards.sol#L21-L23](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/WithRewards.sol#L21-L23)

```diff
-  function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {
+  function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool status_) {
-    return interfaceId == type(IWithRewards).interfaceId || interfaceId == type(IAdapter).interfaceId;
+    status_ = interfaceId == type(IWithRewards).interfaceId || interfaceId == type(IAdapter).interfaceId;
  }
```
## Unused custom error
Consider removing the custom error below that has been declared but never used in the contract to save gas on contract deployment:

[File: CloneFactory.sol#L32](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneFactory.sol#L32)

```diff
-  error NotEndorsed(bytes32 templateKey);
```
## `||` costs less gas than its equivalent `&&`
Rule of thumb: `(x && y)` is `(!(!x || !y))`

Even with the 10k Optimizer enabled: `||`, OR conditions cost less than their equivalent `&&`, AND conditions.

As an example, the instance below may be refactored as follows:

[File: PermissionRegistry.sol#L43](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L43)

```diff
-      if (newPermissions[i].endorsed && newPermissions[i].rejected) revert Mismatch();
+      if (!(!newPermissions[i].endorsed || !newPermissions[i].rejected)) revert Mismatch();
```
## Unchecked SafeMath saves gas
"Checked" math, which is default in ^0.8.0 is not free. The compiler will add some overflow checks, somehow similar to those implemented by `SafeMath`. While it is reasonable to expect these checks to be less expensive than the current `SafeMath`, one should keep in mind that these checks will increase the cost of "basic math operation" that were not previously covered. This particularly concerns variable increments in for loops. When no arithmetic overflow/underflow is going to happen, `unchecked { ++i ;}` to use the previous wrapping behavior further saves gas just as in the for loop below as an example:

[File: PermissionRegistry.sol#L42-L48](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L42-L48)

```diff
-        for (uint256 i = 0; i < len; i++) {
+        for (uint256 i = 0; i < len;) {
              if (newPermissions[i].endorsed && newPermissions[i].rejected) revert Mismatch();

              [ ... ]

+            unchecked {
+                i++;
+            }
          }
```
## Non-strict inequalities are cheaper than strict ones
In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).

As an example, consider replacing `>=` with the strict counterpart `>` in the following inequality instance:

[File: YearnAdapter.sol#L150](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L150)

```diff
-        if (assets >= _depositLimit) return 0;
// Rationale for subtracting 1 on the right side of the inequality:
// x >= 10; [10, 11, 12, ...]
// x > 10 - 1 is the same as x > 9; [10, 11, 12 ...]
+        if (assets > _depositLimit - 1) return 0;
```
## += and -= cost more gas
`+=` and `-=` generally cost 22 more gas than writing out the assigned equation explicitly. The amount of gas wasted can be quite sizable when repeatedly operated in a loop.

For instance, the `+=` instance below may be refactored as follows:

[File: MultiRewardStaking.sol#L408](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L408)

```diff
-    rewardInfos[_rewardToken].index += deltaIndex;
+    rewardInfos[_rewardToken].index = rewardInfos[_rewardToken].index + deltaIndex;
```
Similarly, the `-=` instance below may be refactored as follows:

[File: MultiRewardEscrow.sol#L110](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L110)

```diff
-      amount -= fee;
+      amount = amount - fee;
```
## Use storage instead of memory for structs/arrays
A storage pointer is cheaper since copying a state struct in memory would incur as many SLOADs and MSTOREs as there are slots. In another words, this causes all fields of the struct/array to be read from storage, incurring a Gcoldsload (2100 gas) for each field of the struct/array, and then further incurring an additional MLOAD rather than a cheap stack read. As such, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables will be much cheaper, involving only Gcoldsload for all associated field reads. Read the whole struct/array into a memory variable only when it is being returned by the function, passed into a function that requires memory, or if the array/struct is being read from another memory array/struct.

For instance, the specific example below may be refactored as follows:

[File: MultiRewardEscrow.sol#L157](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L157)

```diff
-      Escrow memory escrow = escrows[escrowId];
+      Escrow storage escrow = escrows[escrowId];
```
## `break` or `continue` in for loop
`for` loop entailing large array with reverting logic should incorporate `break` or `continue` to cater for element(s) failing to get through the iteration(s). This will tremendously save gas on instances where the loop specifically fails to execute at the end of the iterations.

Here is an instance entailed where `setFees()` is associated with reverting `DontGetGreedy()`:

[File: MultiRewardEscrow.sol#L207-L216](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L207-L216)

```solidity
  function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {
    if (tokens.length != tokenFees.length) revert ArraysNotMatching(tokens.length, tokenFees.length);

    for (uint256 i = 0; i < tokens.length; i++) {
      if (tokenFees[i] >= 1e17) revert DontGetGreedy(tokenFees[i]);

      fees[tokens[i]].feePerc = tokenFees[i];
      emit FeeSet(tokens[i], tokenFees[i]);
    }
  }
```
## Private function with embedded modifier reduces contract size
Consider having the logic of a modifier embedded through a private function to reduce contract size if need be. A private visibility that saves more gas on function calls than the internal visibility is adopted because the modifier will only be making this call inside the contract.

For instance, the modifier instance below may be refactored as follows:

[File: Vault.sol#L496-L499](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L496-L499)

```diff
+    function _syncFeeCheckpoint() private view {
+        highWaterMark = convertToAssets(1e18);
+    }

    modifier syncFeeCheckpoint() {
        _;
-        highWaterMark = convertToAssets(1e18);
+        _syncFeeCheckpoint();
    }
```
## Function order affects gas consumption
The order of function will also have an impact on gas consumption. Because in smart contracts, there is a difference in the order of the functions. Each position will have an extra 22 gas. The order is dependent on method ID. So, if you rename the frequently accessed function to more early method ID, you can save gas cost. Please visit the following site for further information:

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

## Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.15",
settings: {
  optimizer: {
    enabled: true,
    runs: 1000,
  },
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.
