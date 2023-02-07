### [L-01] `externalRegistry` is not validated in YearnAdapter.initialize


- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L45

```
        yVault = VaultAPI(IYearnRegistry(externalRegistry).latestVault(_asset)); //@audit
```

When `YearnAdapter` is initialized, `yVault` is from `externalRegistry`. But there is no endorsement check about `externalRegistry`. So the creator of the adapter can send an invalid Yearn registry. So it is recommended to validate endorsement of `externalRegistry` to prevent this.

### [L-02] `MultiRewardEscrow.lock` can revert for small fee percents

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L106-L112

```
    uint256 feePerc = fees[token].feePerc;
    if (feePerc > 0) {
      uint256 fee = Math.mulDiv(amount, feePerc, 1e18);

      amount -= fee;
      token.safeTransfer(feeRecipient, fee);
    }

```

When the fee percent is very small, `amount * feePerc` can be less than `1e18` and `fee` will be 0. The transfer of zero amount will revert for some tokens and this will cause the revert of `MultiRewardEscrow.lock`.
It is recommended to use another if conditional before the safeTransfer.

### [L-03] The return type of `VaultRegistry.getSubmitter` is wrong

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L75


`VaultRegistry.getSubmitter` returns `VaultMetadata` in the implementation.
```
function getSubmitter(address vault) external view returns (VaultMetadata memory) {
```

But it should return address as we can see in the interface.
```
function getSubmitter(address vault) external view returns (address);
```

I think the return type and the implementation of `VaultRegistry.getSubmitter` are not correct. `getSubmitter` is not used in the repository so I can not check anymore.


### [L-04] Large decimal tokens can't be used as reward tokens.

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L274


`MultiRewardStaking.addRewardToken` will revert for `decimals > 19` in the following line.
```
uint64 ONE = (10**IERC20Metadata(address(rewardToken)).decimals()).safeCastTo64();
```
So we can't use those tokens as reward tokens.


### [N-01] Typo

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L701

Mismatch is not correct here.
```
if (length1 != length2) revert ArrayLengthMissmatch();
```

### [N-02] Open TODOs
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L516

There is an open TODO for repository for an audit.


