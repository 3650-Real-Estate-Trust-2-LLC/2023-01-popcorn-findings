
##ignores return value by IERC20Upgradeable also Unsafe IERC20 Operation must use (safetransferFrom) instead of (transferFrom) In order to verify if the transfer has not been made :-


 VaultController.addStakingRewardsTokens(address[],bytes[]) (src/vault/VaultController.sol#433-474) ignores return value by IERC20Upgradeable(rewardsToken).transferFrom(msg.sender,address(adminProxy),amount) (src/vault/VaultController.sol#457)

VaultController.fundStakingRewards(address[],IERC20Upgradeable[],uint256[]) (src/vault/VaultController.sol#512-529) ignores return value by rewardTokens[i].transferFrom(msg.sender,address(this),amounts[i]) (src/vault/VaultController.sol#526)
