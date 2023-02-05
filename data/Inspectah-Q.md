# Low and Non-Critical Issues

## Function visibility can be set to view

Instances: 

- VaultController.sol
    
    [L-242](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L242)
    
    ```solidity
    function _encodeAdapterData(DeploymentArgs memory adapterData, bytes memory baseAdapterData internal
        returns (bytes memory)
    ```
    
    [L-667](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L667)
    
    ```solidity
    function _verifyCreatorOrOwner(address vault) internal returns (VaultMetadata memory metadata) {
    ```
    

## Unused parameter

Instances 

- VaultController.sol
    
    [L-390](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L390) 
    
    `address vault` argument is not used in the function. 
    
    ```solidity
    function _registerVault(address vault, VaultMetadata memory metadata) internal {
    ```
    

## Unused local variable

- [Vault.sol](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L448)

```
function accruedPerformanceFee() public view returns (uint256) {
        uint256 highWaterMark_ = highWaterMark;
        uint256 shareValue = convertToAssets(1e18);
        uint256 performanceFee = fees.performance;

        return
            performanceFee > 0 && shareValue > highWaterMark
                ? performanceFee.mulDiv(
                    (shareValue - highWaterMark) * totalSupply(),
                    1e36,
                    Math.Rounding.Down
                )
                : 0;
    }
```

State variable `highWaterMark` is copied to a local variable `highWaterMark_`but the function uses the state variable to read the value.

## Success of delegateCall not checked

- [AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L444)

```solidity
address(strategy).delegatecall(
                abi.encodeWithSignature("harvest()")
            );
        }
emit Harvested();
```

Delegate call returns 2 values. The first one being the success of the transaction and the second is the return data. Success of the delegate call is not checked before emitting the `Harvested` event