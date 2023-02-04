in some vaults, in deposit functions, there is the ability to pause and unpause deposit and withdraw functions.
for example 

https://github.com/beefyfinance/beefy-contracts/blob/18ffce2c4c8f66865636efb92a24f7dc8f258e20/contracts/BIFI/strategies/Aave/StrategyAave.sol#L406
https://github.com/beefyfinance/beefy-contracts/blob/18ffce2c4c8f66865636efb92a24f7dc8f258e20/contracts/BIFI/strategies/Aave/StrategyAave.sol#L412
https://github.com/beefyfinance/beefy-contracts/blob/18ffce2c4c8f66865636efb92a24f7dc8f258e20/contracts/BIFI/strategies/Aave/StrategyAave.sol#L418

this platform is designed to work with other protocols so, in deposit and withdraw, you should track whether other platforms are paused or no.

////////////////////////////////////////////// ***** //////////////////////////////////////////////