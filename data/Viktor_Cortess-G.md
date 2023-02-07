### [G-01] USE NAMED RETURNS WHERE APPROPRIATE (6 INSTANCES)

**src\vault\adapter\beefy\BeefyAdapter.sol**

    167: function previewRedeem(uint256 shares)
        public
        view
        override
        returns (uint256)

    136: function rewardTokens() external view override returns (address[] memory) {

**src\vault\adapter\abstracts\AdapterBase.sol**

     197: ) public virtual override returns (uint256) {
     177: ) public virtual override returns (uint256) {
     129: function mint(uint256 shares, address receiver)
        public
        virtual
        override
        returns (uint256)
     110: function deposit(uint256 assets, address receiver)
        public
        virtual
        override
        returns (uint256)

### [G-02] REVERT() STATEMENTS THAT CHECK INPUT ARGUMENTS SHOULD BE AT THE TOP OF THE FUNCTION

Checks that involve input parameters should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unfortunate case.

**src\utils\MultiRewardStaking.sol**

     295: function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {
    +   if (rewardsPerSecond == 0) revert ZeroAmount();
        RewardInfo memory rewards = rewardInfos[rewardToken];

    -   if (rewardsPerSecond == 0) revert ZeroAmount(); //Move it up
        if (rewards.lastUpdatedTimestamp == 0) revert RewardTokenDoesntExist(rewardToken);
        if (rewards.rewardsPerSecond == 0) revert RewardsAreDynamic(rewardToken);

**src\vault\Vault.sol**

     57: function initialize(
        IERC20 asset_,
        IERC4626 adapter_,
        VaultFees calldata fees_,
        address feeRecipient_,
        address owner
    ) external initializer {
     +   if (address(asset_) == address(0)) revert InvalidAsset();
     +   if (feeRecipient_ == address(0)) revert InvalidFeeRecipient();
     +   if (address(asset_) != adapter_.asset()) revert InvalidAdapter();

        __ERC20_init(
            string.concat(
                "Popcorn ",
                IERC20Metadata(address(asset_)).name(),
                " Vault"
            ),
            string.concat("pop-", IERC20Metadata(address(asset_)).symbol())
        );
        __Owned_init(owner); 

    - if (address(asset_) == address(0)) revert InvalidAsset(); 
    - if (address(asset_) != adapter_.asset()) revert InvalidAdapter();

        asset = asset_;
        adapter = adapter_;

        asset.approve(address(adapter_), type(uint256).max);

        _decimals = IERC20Metadata(address(asset_)).decimals();

        INITIAL_CHAIN_ID = block.chainid;
        INITIAL_DOMAIN_SEPARATOR = computeDomainSeparator();

        feesUpdatedAt = block.timestamp;
        fees = fees_;

    - if (feeRecipient_ == address(0)) revert InvalidFeeRecipient();
        feeRecipient = feeRecipient_;

        contractName = keccak256(
            abi.encodePacked("Popcorn", name(), block.timestamp, "Vault")
        );

        emit VaultInitialized(contractName, address(asset));
    }

**src\vault\VaultController.sol**

If we put `token == address(0)` check in the beginning we can revert earlier in case of `token == address(0)`

**Before: testFail__deployStaking_token_is_addressZero() (gas: 28177)**
**After: testFail__deployStaking_token_is_addressZero() (gas: 16417)**

     679: function _verifyToken(address token) internal view {
    if ( + token == address(0) ||
      (
        permissionRegistry.endorsed(address(0))
          ? !permissionRegistry.endorsed(token)
          : permissionRegistry.rejected(token)
      ) ||
      cloneRegistry.cloneExists(token) ||
    -  token == address(0) //UP
    ) revert NotAllowed(token);
  }


### [GAS-03] State variables should be cached in stack variables rather than re-reading them from storage.

**src\vault\Vault.sol**

There is an unused variable highWaterMark_ in the function accruedPerformanceFee()   that is supposed to be a cached version of highWaterMark.
But in the return statement, we see 2 instances of using the state variable highWaterMark. Probably a typo.

     447: function accruedPerformanceFee() public view returns (uint256) {
        uint256 highWaterMark_ = highWaterMark;
        uint256 shareValue = convertToAssets(1e18);
        uint256 performanceFee = fees.performance;

        return
            performanceFee > 0 && shareValue > highWaterMark 
                ? performanceFee.mulDiv(
                    (shareValue - highWaterMark) * totalSupply(),
                    1e36,
                    Math.Rounding.Down
                )
                : 0;
    }

### [G-04] USAGE OF UINTS/INTS SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations to reduce the element size from 32 bytes to the desired size.

**src\vault\VaultController.sol**

     438: + //uint256 len = vaults.length;
          - uint8 len = uint8(vaults.length);
            for (uint256 i = 0; i < len; i++) {

Tests:

with uint8 len: test__addStakingRewardsTokens() (gas: 3765200)
with uint256 len :test__addStakingRewardsTokens() (gas: 3765176)

    491: uint8 len = uint8(vaults.length);
    497: for (uint256 i = 0; i < len; i++) {

    520: uint8 len = uint8(vaults.length);
    526: for (uint256 i = 0; i < len; i++) {

    566: uint8 len = uint8(templateCategories.length);
    for (uint256 i = 0; i < len; i++) {

    586: uint8 len = uint8(templateCategories.length);
    590: for (uint256 i = 0; i < len; i++) {

    609: uint8 len = uint8(vaults.length);
    for (uint256 i = 0; i < len; i++) {

    622: uint8 len = uint8(vaults.length);
    for (uint256 i = 0; i < len; i++) {

    635: uint8 len = uint8(vaults.length);
    for (uint256 i = 0; i < len; i++) {

    648: uint8 len = uint8(vaults.length);
    for (uint256 i = 0; i < len; i++) {

     768: uint8 len = uint8(adapters.length);
    for (uint256 i = 0; i < len; i++) {

    808: uint8 len = uint8(adapters.length);
    for (uint256 i = 0; i < len; i++) {

### [G-05] COMBINE 2 IF STATEMENTS WITH || AND THE SAME ERROR TO SAVE SOME GAS

**src\vault\VaultController.sol**

     695: function _verifyAdapterConfiguration(address adapter, bytes32 adapterId) internal view {
    if (adapter != address(0)) { 
     - if (adapterId > 0) revert AdapterConfigFaulty();
     - if (!cloneRegistry.cloneExists(adapter)) revert AdapterConfigFaulty();

     + if (adapterId > 0 || !cloneRegistry.cloneExists(adapter)) revert AdapterConfigFaulty();

         }
       }

**src\utils\MultiRewardStaking.sol**

It's also possible to make the same in function changeRewardSpeed() if you are ready to use only one error for both cases

    299: if (rewards.lastUpdatedTimestamp == 0) revert RewardTokenDoesntExist(rewardToken);
            if (rewards.rewardsPerSecond == 0) revert RewardsAreDynamic(rewardToken); 


