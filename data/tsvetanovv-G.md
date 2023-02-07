### GAS Issues List
| Number |Issues Details
|:--:|:-------|:--:|
|[GAS-01]| Unchecking arithmetics operations that can’t underflow/overflow
|[GAS-02]| `x = x - y` costs less gas than `x -= y`, same for addition
|[GAS-03]| State variables only set in the constructor should be declared immutable
***

## [GAS-01] UNCHECKING ARITHMETICS OPERATIONS THAT CAN’T UNDERFLOW/OVERFLOW
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

```
PermissionRegistry.sol
42: for (uint256 i = 0; i < len; i++) {

src/utils/MultiRewardEscrow.sol
155: for (uint256 i = 0; i < escrowIds.length; i++) {
210: for (uint256 i = 0; i < tokens.length; i++) {

src/utils/MultiRewardStaking.sol
171: for (uint8 i; i < _rewardTokens.length; i++) {
373: for (uint8 i; i < _rewardTokens.length; i++) {

src/vault/VaultController.sol 
319: for (uint8 i = 0; i < len; i++) {
337: for (uint8 i = 0; i < len; i++) {
357: for (uint8 i = 0; i < len; i++) {
374: for (uint8 i = 0; i < len; i++) {
437: for (uint256 i = 0; i < len; i++) {
494: for (uint256 i = 0; i < len; i++) {
523: for (uint256 i = 0; i < len; i++) {
564: for (uint256 i = 0; i < len; i++) {
587: for (uint256 i = 0; i < len; i++) {
607: for (uint256 i = 0; i < len; i++) {
620: for (uint256 i = 0; i < len; i++) {
633: for (uint256 i = 0; i < len; i++) {
646: for (uint256 i = 0; i < len; i++) {
766: for (uint256 i = 0; i < len; i++) {
806: for (uint256 i = 0; i < len; i++) {
```
***

## [GAS-02] `x = x - y` costs less gas than `x -= y`, same for addition

```
src/utils/MultiRewardEscrow.sol
110: amount -= fee;
162: escrows[escrowId].balance -= claimable;

src/vault/adapter/beefy/BeefyAdapter.sol
178: assets -= assets.mulDiv(

src/utils/MultiRewardStaking.sol 
357: amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);
431: accruedRewards[_user][_rewardToken] += supplierDelta;

src/vault/Vault.sol
228: shares += feeShares;
343: shares += shares.mulDiv(
365: assets += assets.mulDiv(
387: assets -= assets.mulDiv(
```

You can replace all `-=` and `+=` occurrences to save gas
***

## [GAS-03] STATE VARIABLES ONLY SET IN THE CONSTRUCTOR SHOULD BE DECLARED IMMUTABLE
It allows setting contract-level variables at construction time which gets stored in code rather than storage.

#### USING IMMUTABLE ON VARIABLES THAT ARE ONLY SET IN THE CONSTRUCTOR AND NEVER AFTER
Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD. Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas)
```
src/vault/DeploymentController.sol
23: ICloneFactory public cloneFactory;
24: ICloneRegistry public cloneRegistry;
25: ITemplateRegistry public templateRegistry;

src/utils/MultiRewardEscrow.sol
53: for (uint256 i = 0; i < escrowIds.length; i++) {
```
***