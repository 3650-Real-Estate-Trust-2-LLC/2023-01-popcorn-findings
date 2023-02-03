- in Vault.deposit(uint256, address), the following approach would be more efficient, since it calls convertToShares only once.
```
function deposit(uint256 assets, address receiver)
        public
        nonReentrant
        whenNotPaused
        syncFeeCheckpoint
        returns (uint256 shares)
    {
        if (receiver == address(0)) revert InvalidReceiver();

        // more efficient approach

        // use -> start
        shares = convertToShares(assets);

        uint256 feeShares =  _shares.mulDiv(uint256(fees.deposit), 1e18, Math.Rounding.Down);
        
        if(feeShares > 0) {
            shares -= feeShares;
            _mint(feeRecipient, feeShares);
        }
        // use -> end
        // instead of -> start

        // uint256 feeShares = convertToShares(
        //     assets.mulDiv(uint256(fees.deposit), 1e18, Math.Rounding.Down)
        // );

        // shares = convertToShares(assets) - feeShares;

        // if (feeShares > 0) _mint(feeRecipient, feeShares);

        // instead of -> end

        _mint(receiver, shares);

        asset.safeTransferFrom(msg.sender, address(this), assets);

        adapter.deposit(assets, address(this));

        emit Deposit(msg.sender, receiver, assets, shares);
    }
```
- declare memory and local variables outside of loops to reduce on gas to be consumed on assigning new memory allocation on each loop.