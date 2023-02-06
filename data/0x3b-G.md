# Strings can be concatenated in order to save gas

As of  Solidity v0.8.12 you can now concatenate strings, this method saves gas and optimizes the code.

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L49-L50
   
     _name = string(abi.encodePacked("Staked ", IERC20Metadata(address(_stakingToken)).name()));
     _symbol = string(abi.encodePacked("pst-", IERC20Metadata(address(_stakingToken)).symbol()));

Suggested change:

     _name = string.concat("Staked ", IERC20Metadata(address(_stakingToken)).name());
     _symbol = string.concat("pst-", IERC20Metadata(address(_stakingToken)).symbol());


# Change struct to bool
The struct is of no need, it makes the code easier to understand and operate but wastes gas in this way. The struct can be removed an on it's place can be just a **bool**.

    struct Permission {
    bool endorsed;
    bool rejected;
    }

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IPermissionRegistry.sol#L8-L11


# Make constants private 

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L25

    uint256 constant DEGRADATION_COEFFICIENT = 10**18;
