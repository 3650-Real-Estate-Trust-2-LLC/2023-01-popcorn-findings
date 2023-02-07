### [G-01] Empty blocks should be removed or emit something


#### Impact
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.


#### Findings:
```
src/vault/Vault.sol:L477    {}

src/vault/VaultRegistry.sol:L21  constructor(address _owner) Owned(_owner) {}

src/vault/CloneRegistry.sol:L22  constructor(address _owner) Owned(_owner) {}

src/vault/PermissionRegistry.sol:L20  constructor(address _owner) Owned(_owner) {}

src/vault/TemplateRegistry.sol:L24  constructor(address _owner) Owned(_owner) {}

src/vault/AdminProxy.sol:L16  constructor(address _owner) Owned(_owner) {}

src/vault/CloneFactory.sol:L23  constructor(address _owner) Owned(_owner) {}
```



### [G-02] ```X += Y``` costs more gas than ```X = X + Y``` for state variables.


#### Impact
Consider changing ```X += Y``` to ```X = X + Y``` to save gas.


#### Findings:
```
src/utils/MultiRewardEscrow.sol:L110      amount -= fee;

src/utils/MultiRewardEscrow.sol:L162      escrows[escrowId].balance -= claimable;

src/utils/MultiRewardStaking.sol:L357      amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);

src/utils/MultiRewardStaking.sol:L408    rewardInfos[_rewardToken].index += deltaIndex;

src/utils/MultiRewardStaking.sol:L431    accruedRewards[_user][_rewardToken] += supplierDelta;

src/vault/Vault.sol:L228        shares += feeShares;

src/vault/Vault.sol:L343        shares += shares.mulDiv(

src/vault/Vault.sol:L365        assets += assets.mulDiv(

src/vault/Vault.sol:L387        assets -= assets.mulDiv(

src/vault/adapter/beefy/BeefyAdapter.sol:L178            assets -= assets.mulDiv(

src/vault/adapter/abstracts/AdapterBase.sol:L158        underlyingBalance += _underlyingBalance() - underlyingBalance_;

src/vault/adapter/abstracts/AdapterBase.sol:L225            underlyingBalance -= underlyingBalance_ - _underlyingBalance();

```


### [G-03] ```++i```/```i++``` should be ```unchecked{++i}```/```unchecked{i++}``` when it is not possible for them to overflow, as is the case when used in for- and while-loops.


#### Impact
The ```unchecked``` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop.


#### Findings:
```
src/utils/MultiRewardEscrow.sol:L53    for (uint256 i = 0; i < escrowIds.length; i++) {

src/utils/MultiRewardEscrow.sol:L155    for (uint256 i = 0; i < escrowIds.length; i++) {

src/utils/MultiRewardEscrow.sol:L210    for (uint256 i = 0; i < tokens.length; i++) {

src/utils/MultiRewardStaking.sol:L171    for (uint8 i; i < _rewardTokens.length; i++) {

src/utils/MultiRewardStaking.sol:L373    for (uint8 i; i < _rewardTokens.length; i++) {

src/vault/PermissionRegistry.sol:L42    for (uint256 i = 0; i < len; i++) {

src/vault/VaultController.sol:L319    for (uint8 i = 0; i < len; i++) {

src/vault/VaultController.sol:L337    for (uint8 i = 0; i < len; i++) {

src/vault/VaultController.sol:L357    for (uint8 i = 0; i < len; i++) {

src/vault/VaultController.sol:L374    for (uint8 i = 0; i < len; i++) {

src/vault/VaultController.sol:L437    for (uint256 i = 0; i < len; i++) {

src/vault/VaultController.sol:L494    for (uint256 i = 0; i < len; i++) {

src/vault/VaultController.sol:L523    for (uint256 i = 0; i < len; i++) {

src/vault/VaultController.sol:L564    for (uint256 i = 0; i < len; i++) {

src/vault/VaultController.sol:L587    for (uint256 i = 0; i < len; i++) {

src/vault/VaultController.sol:L607    for (uint256 i = 0; i < len; i++) {

src/vault/VaultController.sol:L620    for (uint256 i = 0; i < len; i++) {

src/vault/VaultController.sol:L633    for (uint256 i = 0; i < len; i++) {

src/vault/VaultController.sol:L646    for (uint256 i = 0; i < len; i++) {

src/vault/VaultController.sol:L766    for (uint256 i = 0; i < len; i++) {

src/vault/VaultController.sol:L806    for (uint256 i = 0; i < len; i++) {

```
