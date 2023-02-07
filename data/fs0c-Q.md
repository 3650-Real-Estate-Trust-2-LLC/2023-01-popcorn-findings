## [QA-01] `changeAdapter` should only be called by owner.

## Impact

Altough there is no immediate threat due to this, but this function should only be called by owner. 

### POC

```solidity
function changeAdapter() external takeFees {
        if (block.timestamp < proposedAdapterTime + quitPeriod)
            revert NotPassedQuitPeriod(quitPeriod);

        adapter.redeem(
            adapter.balanceOf(address(this)),
            address(this),
            address(this)
        );

        asset.approve(address(adapter), 0);

        emit ChangedAdapter(adapter, proposedAdapter);

        adapter = proposedAdapter;

        asset.approve(address(adapter), type(uint256).max);

        adapter.deposit(asset.balanceOf(address(this)), address(this));
    }
```

### Recommendation

Add `onlyOwner` modfier to the function.

## [QA-02] Remove the unnecessary variable `assetsCheckpoint` and change the natspec comments regarding that.

The unnecessary variable `assetsCheckpoint` can be removed and the natspec comment of `changeAdapter` has to be modified to remove the variable.

```solidity
* @notice Set a new Adapter for this Vault after the quit period has passed.
     * @dev This migration function will remove all assets from the old Vault and move them into the new vault
     * @dev Additionally it will zero old allowances and set new ones
     * @dev Last we update HWM and assetsCheckpoint for fees to make sure they adjust to the new adapter
     */
    function changeAdapter() external takeFees {
```