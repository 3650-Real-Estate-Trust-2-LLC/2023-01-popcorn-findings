# Gas Report
## Finding Summary

1. [**uint256 Iterator Checked Each Iteration**](#1-uint256-Iterator-Checked-Each-Iteration)
2. [**x += y Used For State Variables**](#2-x--y-Used-For-State-Variables)
3. [**x -= y Used For State Variables**](#3-x---y-Used-For-State-Variables)
4. [**Use Unused Cache Variable**](#4-Use-Unused-Cache-Variable)
5. [**Unchecked Arithmetic**](#5-Unchecked-Arithmetic)
6. [**Add view Modifer**](#6-Add-view-Modifer)
7. [**Short Circuit**](#7-Short-Circuit)

## 1. uint256 Iterator Checked Each Iteration

A `uint256` iterator will not overflow before the check variable overflows. Unchecking the iterator increment saves gas.

**Example**
```solidity
//From
for (uint256 i; i < len; ++i) {
	//Do Something
}
//To
for (uint256 i; i < len;) {
	//Do Something
	unchecked { ++i; }
}
```

*/src/vault/PermissionRegistry.sol*

```solidity
42:	for (uint256 i = 0; i < len; i++) {
```

*/src/utils/MultiRewardEscrow.sol*

```solidity
53:	for (uint256 i = 0; i < escrowIds.length; i++) {
155:	for (uint256 i = 0; i < escrowIds.length; i++) {
210:	for (uint256 i = 0; i < tokens.length; i++) {
```

*/src/vault/VaultController.sol*

```solidity
437:	for (uint256 i = 0; i < len; i++) {
494:	for (uint256 i = 0; i < len; i++) {
523:	for (uint256 i = 0; i < len; i++) {
564:	for (uint256 i = 0; i < len; i++) {
587:	for (uint256 i = 0; i < len; i++) {
607:	for (uint256 i = 0; i < len; i++) {
620:	for (uint256 i = 0; i < len; i++) {
633:	for (uint256 i = 0; i < len; i++) {
646:	for (uint256 i = 0; i < len; i++) {
766:	for (uint256 i = 0; i < len; i++) {
806:	for (uint256 i = 0; i < len; i++) {
```

## 2. x += y Used For State Variables

When dealing with state variables `x = x + y` is cheaper than `x += y`.

*/src/utils/MultiRewardStaking.sol*

```solidity
408:	rewardInfos[_rewardToken].index += deltaIndex;
```

## 3. x -= y Used For State Variables

When dealing with state variables `x = x - y` is cheaper than `x -= y`.

*/src/utils/MultiRewardEscrow.sol*

```solidity
162:	escrows[escrowId].balance -= claimable;
```

## 4. Use Unused Cache Variable

There is cache variable [`highWaterMark_`](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L448) not used (`accruedPerformanceFee` returns `highWaterMark` not `highWaterMark_`). Consider fixing this issue to save gas.

## 5. Unchecked Arithmetic

Modern Solidity will automatically revert if a value overflows/underflows. More computation is used in order to force an overflow/underflow revert by default. When overflows/underflows are not possible (likely due to an earlier check), code can be placed inside an `unchecked` brace to save gas.

```solidity
unchecked {
	//Code
}
```

The findings below display the first line which allows the second line to go `unchecked`. ONLY the bolded parts (\*\*CODE\*\*) of the second line are suggested to be put in an `unchecked` brace.

*/src/vault/adapter/yearn/YearnAdapter.sol*

```solidity
150:	if (assets >= _depositLimit) return 0;
151:	return **_depositLimit - assets**;
```

*/src/utils/MultiRewardStaking.sol*

```solidity
356:	if (rewardsEndTimestamp > block.timestamp)
357:	amount += uint256(rewardsPerSecond) * (**rewardsEndTimestamp - block.timestamp**);
```

## 6. Add view Modifer

Cosider adding a `view` modifier to [`_calcRewardsEnd`](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L351) to save gas.

## 7. Short Circuit

The `deploy` function of [CloneFactory.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneFactory.sol) uses a success variable, which only has an effect if `(template.requiresInitData)`. 

Consider implementing the following code to save gas: 

```solidity
function deploy(Template calldata template, bytes calldata data) external onlyOwner returns (address clone) {
    clone = Clones.clone(template.implementation);

    if (template.requiresInitData) {
        (bool success, ) = clone.call(data);
        if (!success) revert DeploymentInitFailed();
    } 

    emit Deployment(clone);
}
```