# State variables only set in the constructor should be declared immutable

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L191
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L31-L33
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L21-L24
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L24-L29

# refactor the code 
refactor the function [MultiRewardStaking._accrueRewards](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L402-L410)
to 

    function _accrueRewards(IERC20 _rewardToken, uint256 accrued) internal {
        uint256 supplyTokens = totalSupply();
        uint224 deltaIndex;
        if (supplyTokens != 0){
            deltaIndex = accrued.mulDiv(uint256(10**decimals()), supplyTokens, Math.Rounding.Down).safeCastTo224();
            rewardInfos[_rewardToken].index += deltaIndex;
        }

        // in the orginal code, if supplyTokens== 0, then deltaIndex equals 0,
        // so it's useless to rewardInfos[_rewardToken].index += 0
        rewardInfos[_rewardToken].lastUpdatedTimestamp = block.timestamp.safeCastTo32();
    }

# refactor the code
refactor the code [Vault.accruedPerformanceFee](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L447-L460) to

    function accruedPerformanceFee() public view returns (uint256) {
        uint256 highWaterMark_ = highWaterMark;
        uint256 shareValue = convertToAssets(1e18);
        uint256 performanceFee = fees.performance;

        return
            // use highWaterMark_ instead of highWaterMark
            performanceFee > 0 && shareValue > highWaterMark_
                ? performanceFee.mulDiv(
                    (shareValue - highWaterMark_) * totalSupply(),
                    1e36,
                    Math.Rounding.Down
                )
                : 0;
    }

# <x> += <y> costs more gas than <x> = <x> + <y> for state variables
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L158
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L225