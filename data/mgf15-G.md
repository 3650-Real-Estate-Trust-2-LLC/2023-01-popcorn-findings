Gas optimizing 

# 1. ++i costs less gas than i++ 

change i++ to ++i to save gas ++i use less gas . 

# 2. Local variable assignment 

Catch frequently used storage variables in memory/stack, converting multiple SLOAD into 1 SLOAD. 

Before:
```
uint256 length = 10;

function loop() public {
    for (uint256 i = 0; i < length; i++) {
        // do something here
    }
}
```
After:
```
uint256 length = 10;

function loop() {
    uint256 l = length;

    for (uint256 i = 0; i < l; ++i) {
        // do something here
    }
}
```

MultiRewardEscrow.sol#L51  
```
    for (uint256 i = 0; i < escrowIds.length; i++) { // here 
  }
```
line 155
```
function claimRewards(bytes32[] memory escrowIds) external {
    for (uint256 i = 0; i < escrowIds.length; i++) { // here
```
line 210
```
    for (uint256 i = 0; i < tokens.length; i++) { // here
```
MultiRewardStaking.sol#L171
```
    for (uint8 i; i < _rewardTokens.length; i++) { // here
```
line 373
```
    for (uint8 i; i < _rewardTokens.length; i++) { // here
```

# 3. Use indexed events as they are less costly compared to non-indexed ones 

MultiRewardStaking.sol#L159
```
  event RewardsClaimed(address indexed user, IERC20 rewardToken, uint256 amount, bool escrowed);
```
line
220
```
  event RewardInfoUpdate(IERC20 rewardToken, uint160 rewardsPerSecond, uint32 rewardsEndTimestamp);

```
Vault.sol#114

```
    event Withdraw(
        address indexed caller,
        address indexed receiver,
        address indexed owner,
        uint256 assets,
        uint256 shares
    );
```
line 512
```
    event NewFeesProposed(VaultFees newFees, uint256 timestamp);
    event ChangedFees(VaultFees oldFees, VaultFees newFees);
    event FeeRecipientUpdated(address oldFeeRecipient, address newFeeRecipient);
```
line 569
```
    event NewAdapterProposed(IERC4626 newAdapter, uint256 timestamp);
    event ChangedAdapter(IERC4626 oldAdapter, IERC4626 newAdapter);
```
line 621
```
    event QuitPeriodSet(uint256 quitPeriod);
```
VaultController.sol#L741
```
  event PerformanceFeeChanged(uint256 oldFee, uint256 newFee);
```
line 781

```
  event HarvestCooldownChanged(uint256 oldCooldown, uint256 newCooldown);
```
line 824
```
  event DeploymentControllerChanged(address oldController, address newController);
```
line 853
```
  event ActiveTemplateIdChanged(bytes32 oldKey, bytes32 newKey);
```
PermissionRegistry.sol#L28
```
  event PermissionSet(address target, bool newEndorsement, bool newRejection);
```