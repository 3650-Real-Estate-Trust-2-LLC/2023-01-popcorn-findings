## [01-GAS] Use external instead of public for functions not called within the contract.
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L632-L640
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L664-L672
- It's recommended that changing these to externally facing contracts as they are not used within the contract itself. This will help in saving gas as public functions cost `496` gas units while an externally facing function only uses `261` gas units. This is because public functions write arguments to memory so that they can be called internally. On the other hand for external functions, the compiler doesn't allow internal calls so its arguments are read from calldata skipping the entire copy step. 


## [02-GAS] Using `calldata` instead of `memory` for read-only arguments in the `initialize` function for `BeefyAdapter` (external) can save gas.
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L47
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L49
- When a function with parameters in memory are called, `abi.decode()` has to use a loop to copy the indexes of calldata into memory indexes. Each iteration of this loop will cost 60 gas units. Consider using calldata directly as this will remove the need for a loop in the contract as well as runtime execution. 



## [03-GAS] Simple internal functions only called once could be reimplemented inline to save gas.
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L479-L483
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L83
- Functions which are implemented in a simple fashion could be moved inline as this may cost ~20 extra gas units because of two extra JUMP instructions.





## [04-GAS] Consider using an assembly check for checking zero addresses.
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L209
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L198
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L82
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L693
- https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L83
- This will save 6 gas units for every instance.





## [05-GAS] Use `++i` instead of `i++` when iterating with for loops. 
- `++i` generally costs less gas than `i++` or `i = i + 1` (about 5 units per increment) because `i++` must increment a value and then "return" the old value which means the program may need to hold two numbers in memory. When `++i` is used, it will only ever use one number in memory. Consider using `++i` where for loops are used for instance  `for(uint256 i = 0; i < len; ++i) {}`. 



