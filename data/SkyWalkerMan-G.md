### Use ***Bytes32*** instead of ***string*** type:

>the string type basically equal to bytes

Instances (4):
``` 
File:src\utils\MultiRewardStaking.sol

11: string private _name;
12: string private _symbol;
```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L31)

``` 
File:src\utils\MultiRewardStaking.sol

59: function name() public view override(ERC20Upgradeable, IERC20Metadata) returns (string memory) {
         return _name;
    }

63: function symbol() public view override(ERC20Upgradeable, IERC20Metadata) returns (string memory) {
         return _symbol;
    }
```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L59)
