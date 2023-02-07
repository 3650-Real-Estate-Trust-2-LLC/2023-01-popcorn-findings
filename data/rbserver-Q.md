## [01] FEE-ON-TRANSFER TOKENS ARE NOT SUPPORTED
This protocol currently does not support fee-on-transfer tokens. For example, for a fee-on-transfer token, calling the following `Vault.deposit` function with the `assets` input being 1e18 can transfer less than 1e18 of such token to the `Vault` contract since the transfer fee is deducted during the transfer; then, when this function futher calls the `AdapterBase.deposit` function with the `assets` input still being 1e18, `IERC20(asset()).safeTransferFrom(caller, address(this), assets)` is executed but this execution reverts because the `Vault` contract does not have sufficient balance of such token for transferring 1e18 again to the `AdapterBase` contract. To support fee-on-transfer tokens, functions like `Vault.deposit` function, which needs to transfer the received assets out again, can be updated to calculate the actual token amount that is received and then transfer out that received amount instead of the `assets` input value.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L134-L158
```solidity
    function deposit(uint256 assets, address receiver)
        public
        nonReentrant
        whenNotPaused
        syncFeeCheckpoint
        returns (uint256 shares)
    {
        ...
        asset.safeTransferFrom(msg.sender, address(this), assets);

        adapter.deposit(assets, address(this));
        ...
    }
```

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L110-L122
```solidity
    function deposit(uint256 assets, address receiver)
        public
        virtual
        override
        returns (uint256)
    {
        ...
        _deposit(_msgSender(), receiver, assets, shares);
        ...
    }
```

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L147-L165
```solidity
    function _deposit(
        address caller,
        address receiver,
        uint256 assets,
        uint256 shares
    ) internal nonReentrant virtual override {
        IERC20(asset()).safeTransferFrom(caller, address(this), assets);
        ...
    }
```

## [02] RETURN VALUE OF DELEGATECALLING STRATEGY'S `harvest` FUNCTION IS NOT CHECKED IN `AdapterBase.harvest` FUNCTION
In the following `AdapterBase.harvest` function, the return value of executing `address(strategy).delegatecall(abi.encodeWithSignature("harvest()"))` is not checked. If executing the strategy's `harvest` function fails, this failure will go unnoticed while emitting the `Harvested` event incorrectly. Moreover, if executing the strategy's `harvest` function fails, such execution might need to be tried again when calling the `AdapterBase.harvest` function next time so the protocol probably should not go into the harvest cooldown. In other words, if needing to set `lastHarvest` to `block.timestamp` after executing the strategy's `harvest` function, it needs to check if such execution is successful or not; if not, the `lastHarvest` probably should not be set to `block.timestamp` to delay the harvest cooldown, and the `Harvested` event should not be emitted.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L438-L450
```solidity
    function harvest() public takeFees {
        if (
            address(strategy) != address(0) &&
            ((lastHarvest + harvestCooldown) < block.timestamp)
        ) {
            // solhint-disable
            address(strategy).delegatecall(
                abi.encodeWithSignature("harvest()")
            );
        }

        emit Harvested();
    }
```

## [03] `YearnAdapter.convertToUnderlyingShares` DOES NOT RETURN `shares` WHEN `totalSupply()` IS 0
When calling the following `YearnAdapter.convertToUnderlyingShares` function, it does not return `shares` when `totalSupply()` is 0, which is different than the `BeefyAdapter.convertToUnderlyingShares` function below that returns `shares` when `totalSupply()` is 0. For consistency with the `BeefyAdapter.convertToUnderlyingShares` function, and to avoid reverting the `YearnAdapter.convertToUnderlyingShares` function call when `totalSupply()` is 0, please consider updating the `YearnAdapter.convertToUnderlyingShares` function to return `shares` when `totalSupply()` is 0.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L129-L137
```solidity
    function convertToUnderlyingShares(uint256, uint256 shares)
        public
        view
        override
        returns (uint256)
    {
        return
            shares.mulDiv(underlyingBalance, totalSupply(), Math.Rounding.Up);
    }
```

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L122-L133
```solidity
    function convertToUnderlyingShares(uint256, uint256 shares)
        public
        view
        override
        returns (uint256)
    {
        uint256 supply = totalSupply();
        return
            supply == 0
                ? shares
                : shares.mulDiv(underlyingBalance, supply, Math.Rounding.Up);
    }
```

## [04] VALUES OF `fees_` ARE NOT CHECKED IN `Vault.initialize` FUNCTION
When calling the following `Vault.initialize` function, the values of `fees_` are not checked. It is possible that these fees are set to be above 1e18 when calling the `Vault.initialize` function; if this happens, calling functions like `Vault.deposit` can revert because the fee amount would exceed the asset amount. To avoid reverting these function calls, please consider checking the values of `fees_` in the `Vault.initialize` function to ensure that these values are capped by some reasonable limits.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L57-L98
```solidity
    function initialize(
        IERC20 asset_,
        IERC4626 adapter_,
        VaultFees calldata fees_,
        address feeRecipient_,
        address owner
    ) external initializer {
        ...
        fees = fees_;
        ...
    }
```

## [05] LIMITS FOR CALLING `Vault.proposeFees` ARE INCONSISTENT WITH LIMIT FOR CALLING `AdapterBase.setPerformanceFee` FUNCTION
The limits for calling the following `Vault.proposeFees` function are 1e18, including the limit used for the vault's performance fee, but the limit for calling the `AdapterBase.setPerformanceFee` function is 2e17. Besides that it is inconsistent and can be confusing to have different limits on the fees, setting the deposit and withdrawal fees to 1e18 means that all asset amounts are considered as fees, which does not make too much sense and is not user-friendly. Hence, please consider using a consistent reasonable limit on the fees for both the `Vault.proposeFees` and `AdapterBase.setPerformanceFee` functions.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L525-L537
```solidity
    function proposeFees(VaultFees calldata newFees) external onlyOwner {
        if (
            newFees.deposit >= 1e18 ||
            newFees.withdrawal >= 1e18 ||
            newFees.management >= 1e18 ||
            newFees.performance >= 1e18
        ) revert InvalidVaultFees();

        proposedFees = newFees;
        proposedFeeTime = block.timestamp;

        emit NewFeesProposed(newFees, block.timestamp);
    }
```

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L549-L556
```solidity
    function setPerformanceFee(uint256 newFee) public onlyOwner {
        // Dont take more than 20% performanceFee
        if (newFee > 2e17) revert InvalidPerformanceFee(newFee);

        emit PerformanceFeeChanged(performanceFee, newFee);

        performanceFee = newFee;
    }
```

## [06] UNSAFE `decimals()` CALLS FOR TOKENS THAT DO NOT IMPLEMENT `decimals()`
Each of the following code calls the relevant token's `decimals()`. According to https://eips.ethereum.org/EIPS/eip-20,  `decimals()` is optional so it is possible that the relevant token does not implement `decimals()`. When this occurs, calling such token's `decimals()` reverts. To support such tokens, helper functions like BoringCrypto's [`safeDecimals`](https://github.com/boringcrypto/BoringSolidity/blob/ccb743d4c3363ca37491b87c6c9b24b1f5fa25dc/contracts/libraries/BoringERC20.sol#L52-L55) can be used instead of directly calling `decimals()` in the code.

```solidity
utils\MultiRewardStaking.sol
  51: _decimals = IERC20Metadata(address(_stakingToken)).decimals(); 
  274: uint64 ONE = (10**IERC20Metadata(address(rewardToken)).decimals()).safeCastTo64(); 

vault\Vault.sol
  82: _decimals = IERC20Metadata(address(asset_)).decimals(); 

vault\adapter\abstracts\AdapterBase.sol
  77: _decimals = IERC20Metadata(asset).decimals(); 

vault\strategy\Pool2SingleAssetCompounder.sol
  27: uint256[] memory amountsOut = IUniswapRouterV2(router).getAmountsOut(ERC20(asset).decimals() ** 10, tradePath); 
```

## [07] UNSAFE `symbol()` CALLS FOR TOKENS THAT HAVE `bytes32` SYMBOLS
Each of the following code calls the relevant token's `symbol()` using `IERC20Metadata`, which expects the token to have a `string` symbol. However, there are legitimate tokens with `bytes32` symbols, such as [MKR](https://etherscan.io/address/0x9f8f72aa9304c8b593d555f12ef6589cc3a579a2#code#L406), and calling their `symbol()` using `IERC20Metadata` will revert. To support such tokens, helper functions like BoringCrypto's [`safeSymbol`](https://github.com/boringcrypto/BoringSolidity/blob/ccb743d4c3363ca37491b87c6c9b24b1f5fa25dc/contracts/libraries/BoringERC20.sol#L36-L39) can be used instead of directly calling `symbol()`  using `IERC20Metadata` in the code below.

```solidity
vault\Vault.sol
  70: string.concat("pop-", IERC20Metadata(address(asset_)).symbol())

vault\adapter\beefy\BeefyAdapter.sol
  71: _symbol = string.concat("popB-", IERC20Metadata(asset()).symbol());

vault\adapter\yearn\YearnAdapter.sol
  52: _symbol = string.concat("popY-", IERC20Metadata(asset()).symbol());
```

## [08] UNSAFE `name()` CALLS FOR TOKENS THAT HAVE `bytes32` NAMES
Each of the following code calls the relevant token's `name()` using `IERC20Metadata`, which expects the token to have a `string` name. However, there are legitimate tokens with `bytes32` names, such as [MKR](https://etherscan.io/address/0x9f8f72aa9304c8b593d555f12ef6589cc3a579a2#code#L473), and calling their `name()` using `IERC20Metadata` will revert. To support such tokens, helper functions like BoringCrypto's [`safeName`](https://github.com/boringcrypto/BoringSolidity/blob/ccb743d4c3363ca37491b87c6c9b24b1f5fa25dc/contracts/libraries/BoringERC20.sol#L44-L47) can be used instead of directly calling `name()`  using `IERC20Metadata` in the following code.

```solidity
vault\Vault.sol
  67: IERC20Metadata(address(asset_)).name(), 

vault\adapter\beefy\BeefyAdapter.sol
  68: IERC20Metadata(asset()).name(), 

vault\adapter\yearn\YearnAdapter.sol
  49: IERC20Metadata(asset()).name(), 
```

## [09] MISSING `address(0)` CHECKS FOR CRITICAL ADDRESS INPUTS
( Please note that the following instances are not found in https://gist.github.com/Picodes/b3c3bc8df01afc2b649534da8007af88#nc-1-missing-checks-for-address0-when-assigning-values-to-address-state-variables. )

To prevent unintended behaviors, critical address inputs should be checked against `address(0)`. `address(0)` checks are missing for `receiver` in the following functions. Please consider checking them.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L110-L122
```solidity
    function deposit(uint256 assets, address receiver)
        public
        virtual
        override
        returns (uint256)
    {
        ...
    }
```

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L129-L141
```solidity
    function mint(uint256 shares, address receiver)
        public
        virtual
        override
        returns (uint256)
    {
        ...
    }
```

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L173-L185
```solidity
    function withdraw(
        uint256 assets,
        address receiver,
        address owner
    ) public virtual override returns (uint256) {
        ...
    }
```

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L193-L204
```solidity
    function redeem(
        uint256 shares,
        address receiver,
        address owner
    ) public virtual override returns (uint256) {
        ...
    }
```

## [10] INCORRECT COMMENTS
The comments of the following functions all mention that "caller must be owner". However, these functions all use the `canCreate` modifier so the allowed users, who are not the owner, can also call these functions. To prevent misleading information, please consider updating the comments of these functions accordingly.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L704-L711
```solidity
  modifier canCreate() {
    if (
      permissionRegistry.endorsed(address(1))
        ? !permissionRegistry.endorsed(msg.sender)
        : permissionRegistry.rejected(msg.sender)
    ) revert NotAllowed(msg.sender);
    _;
  }
```

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L79-L117
```solidity
   * @notice Deploy a new Vault. Optionally with an Adapter and Staking. Caller must be owner.
   ...
  function deployVault(
    VaultInitParams memory vaultData,
    DeploymentArgs memory adapterData,
    DeploymentArgs memory strategyData,
    address staking,
    bytes memory rewardsData,
    VaultMetadata memory metadata,
    uint256 initialDeposit
  ) external canCreate returns (address vault) {
    ...
  }
```

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L181-L197
```solidity
   * @notice Deploy a new Adapter with our without a strategy. Caller must be owner.
   ...
  function deployAdapter(
    IERC20 asset,
    DeploymentArgs memory adapterData,
    DeploymentArgs memory strategyData,
    uint256 initialDeposit
  ) external canCreate returns (address adapter) {
    ...
  }
```

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L274-L281
```solidity
   * @notice Deploy a new staking contract. Caller must be owner.
   ...
  function deployStaking(IERC20 asset) external canCreate returns (address) {
    ...
  }
```

## [11] UNRESOLVED TODO COMMENT
Comments regarding todos indicate that there are unresolved action items for implementation, which need to be addressed before the protocol deployment. Please review the todo comment in the following code.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L516-L517
```solidity
    // TODO use deterministic fee recipient proxy
    address FEE_RECIPIENT = address(0x4444);
```

## [12] COMMENT THAT IS NOT EXACT
The following code comment mentions `X seconds` but the code uses `1 days`. To avoid ambiguity, please consider updating the comment to reflect `1 days`.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L501-L502
```solidity
        // Dont wait more than X seconds
        if (newCooldown >= 1 days) revert InvalidHarvestCooldown(newCooldown);
```

## [13] WORD TYPING TYPO
`aftwards` can be `afterwards` in the following code comment.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L477
```solidity
     * @dev It aftwards sets up anything required by the strategy to call `harvest()` like approvals etc.
```

## [14] REDUNDANT NAMED RETURNS
When a function has unused named returns and used return statement, these named returns become redundant. To improve readability and maintainability, these variables for the named returns can be removed while keeping the return statement for the following `execute` function.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/AdminProxy.sol#L19-L25
```solidity
  function execute(address target, bytes calldata callData)
    external
    onlyOwner
    returns (bool success, bytes memory returndata)
  {
    return target.call(callData);
  }
```

## [15] CONSTANTS CAN BE USED INSTEAD OF MAGIC NUMBERS
To improve readability and maintainability, a constant can be used instead of the magic number. Please consider replacing the magic numbers, such as `1e18`, used in the following code with constants.

```solidity
vault\Vault.sol
  144: assets.mulDiv(uint256(fees.deposit), 1e18, Math.Rounding.Down)  
  183: 1e18 - depositFee,  
  224: 1e18 - withdrawalFee,   
  330: assets.mulDiv(uint256(fees.deposit), 1e18, Math.Rounding.Down)  
  345: 1e18 - depositFee,  
  437: ) / 1e18    
  449: uint256 shareValue = convertToAssets(1e18); 
  484: uint256 shareValue = convertToAssets(1e18);  
  498: highWaterMark = convertToAssets(1e18);  
  527: newFees.deposit >= 1e18 ||  
  528: newFees.withdrawal >= 1e18 ||  
  529: newFees.management >= 1e18 ||  
  530: newFees.performance >= 1e18  

vault\adapter\abstracts\AdapterBase.sol
  85: highWaterMark = 1e18;   
  531: uint256 shareValue = convertToAssets(1e18); 
  538: 1e36,   
  551: if (newFee > 2e17) revert InvalidPerformanceFee(newFee);    
  565: uint256 shareValue = convertToAssets(1e18); 
```

## [16] FLOATING PRAGMAS
It is a best practice to lock pragmas instead of using floating pragmas to ensure that contracts are tested and deployed with the intended compiler version. Accidentally deploying contracts with different compiler versions can lead to unexpected risks and undiscovered bugs. Please consider locking pragmas for the following files.

```solidity
utils\MultiRewardEscrow.sol
  4: pragma solidity ^0.8.15;

utils\MultiRewardStaking.sol
  4: pragma solidity ^0.8.15;

vault\AdminProxy.sol
  4: pragma solidity ^0.8.15;

vault\CloneFactory.sol
  4: pragma solidity ^0.8.15;

vault\CloneRegistry.sol
  4: pragma solidity ^0.8.15;

vault\DeploymentController.sol
  4: pragma solidity ^0.8.15;

vault\PermissionRegistry.sol
  4: pragma solidity ^0.8.15;

vault\TemplateRegistry.sol
  4: pragma solidity ^0.8.15;

vault\Vault.sol
  4: pragma solidity ^0.8.15;

vault\VaultController.sol
  3: pragma solidity ^0.8.15;

vault\VaultRegistry.sol
  4: pragma solidity ^0.8.15;

vault\adapter\abstracts\AdapterBase.sol
  4: pragma solidity ^0.8.15;

vault\adapter\abstracts\OnlyStrategy.sol
  4: pragma solidity ^0.8.15;

vault\adapter\abstracts\WithRewards.sol
  4: pragma solidity ^0.8.15;

vault\adapter\beefy\BeefyAdapter.sol
  4: pragma solidity ^0.8.15;

vault\adapter\yearn\YearnAdapter.sol
  4: pragma solidity ^0.8.15;
```

## [17] INCOMPLETE NATSPEC COMMENTS
NatSpec comments provide rich code documentation. The following functions miss the `@param` and/or `@return` comments. Please consider completing the NatSpec comments for these functions.

```solidity
vault\adapter\abstracts\AdapterBase.sol
  110: function deposit(uint256 assets, address receiver)  
  147: function _deposit(  
  173: function withdraw(  
  294: function _previewDeposit(uint256 assets)    
  343: function _previewWithdraw(uint256 assets)   
  380: function _convertToShares(uint256 assets, Math.Rounding rounding)   
  404: function maxDeposit(address)    
  479: function _verifyAndSetupStrategy(bytes4[8] memory requiredSigs) internal {  

vault\adapter\yearn\YearnAdapter.sol
  80: function _totalAssets() internal view override returns (uint256) {  
  89: function _shareValue(uint256 yShares) internal view returns (uint256) {  
  101: function _freeFunds() internal view returns (uint256) { 
  109: function _yTotalAssets() internal view returns (uint256) { 
  114: function _calculateLockedProfit() internal view returns (uint256) { 
  129: function convertToUnderlyingShares(uint256, uint256 shares)  
```

## [18] MISSING NATSPEC COMMENTS
NatSpec comments provide rich code documentation. The following functions miss NatSpec comments. Please consider adding NatSpec comments for these functions.

```solidity
vault\PermissionRegistry.sol
  51: function endorsed(address target) external view returns (bool) {  
  55: function rejected(address target) external view returns (bool) {  

vault\Vault.sol
  100: function decimals() public view override returns (uint8) {  
  124: function deposit(uint256 assets) public returns (uint256) { 
  160: function mint(uint256 shares) external returns (uint256) {  
  200: function withdraw(uint256 assets) public returns (uint256) {    
  242: function redeem(uint256 shares) external returns (uint256) {    
  716: function computeDomainSeparator() internal view virtual returns (bytes32) { 

vault\adapter\abstracts\AdapterBase.sol
  684: function computeDomainSeparator() internal view virtual returns (bytes32) { 

vault\adapter\beefy\BeefyAdapter.sol
  108: function _totalAssets() internal view override returns (uint256) {  
  117: function _underlyingBalance() internal view override returns (uint256) {    

vault\adapter\yearn\YearnAdapter.sol
  84: function _underlyingBalance() internal view override returns (uint256) {    
  158: function _protocolDeposit(uint256 amount, uint256)  
  166: function _protocolWithdraw(uint256 assets, uint256 shares)  
```