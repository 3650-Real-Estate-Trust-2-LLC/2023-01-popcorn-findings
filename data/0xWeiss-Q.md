# [L-01] CHANGE VARIBALES TO IMMUTABLE

The following variables, do not have a function to update them and are initialized in the constructor, therefore they should be immutable or constant and hardcode the addresses.

    IBeefyVault public beefyVault;
    IBeefyBooster public beefyBooster;
    IBeefyBalanceCheck public beefyBalanceCheck;

The variables are here:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L27-L29

