# NC-1 

 ## Wrong natspec 

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L368-L381

Here, the Natspec is wrong . 
```
   * @notice Change adapter of a vault to the previously proposed adapter.

```

It should be instead 

```
* @notice Change vault proposed fees to the actual fees.
```
