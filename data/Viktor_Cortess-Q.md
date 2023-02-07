### [NC] Unused variable

`_registerVault()` of `VaultController` has an unused variable, which can be commented out to prevent compilation warnings and improve code readability

    390: function _registerVault(address /* vault */, VaultMetadata memory metadata) internal { 
    (bool success, bytes memory returnData) = adminProxy.execute(
      address(vaultRegistry),
      abi.encodeWithSelector(IVaultRegistry.registerVault.selector, metadata)
    );
    if (!success) revert UnderlyingError(returnData);
    }


### [L-01] ADD ZERO ADDRESS VALIDATION IN CONSTRUCTOR 

The parameters used in the constructor are used to initialize the state variables, errors in these can lead to the redeployment of the contract.

**src\vault\VaultController.sol**

    53: constructor(
    address _owner,
    IAdminProxy _adminProxy,
    IDeploymentController _deploymentController,
    IVaultRegistry _vaultRegistry,
    IPermissionRegistry _permissionRegistry,
    IMultiRewardEscrow _escrow
      ) Owned(_owner) {
    adminProxy = _adminProxy;
    vaultRegistry = _vaultRegistry;
    permissionRegistry = _permissionRegistry;
    escrow = _escrow;

    _setDeploymentController(_deploymentController);

    activeTemplateId[STAKING] = "MultiRewardStaking";
    activeTemplateId[VAULT] = "V1";
    }

**src\utils\MultiRewardEscrow.sol**

     30: constructor(address _owner, address _feeRecipient) Owned(_owner) {
    feeRecipient = _feeRecipient;
    }

**src\vault\DeploymentController.sol**

    35: constructor(
    address _owner,
    ICloneFactory _cloneFactory,
    ICloneRegistry _cloneRegistry,  //@LOW zero values consructor
    ITemplateRegistry _templateRegistry
      ) Owned(_owner) {
    cloneFactory = _cloneFactory;
    cloneRegistry = _cloneRegistry;
    templateRegistry = _templateRegistry;
    }

### [L-02] AVOID SHADOWING INHERITED STATE VARIABLES

**src\vault\Vault.sol**

    72: __Owned_init(owner);
    230: if (msg.sender != owner) 
            _approve(owner, msg.sender, allowance(owner, msg.sender) - shares);

        _burn(owner, shares);

        if (feeShares > 0) _mint(feeRecipient, feeShares);

        adapter.withdraw(assets, receiver, address(this));

        emit Withdraw(msg.sender, receiver, owner, assets, shares);
    271: _burn(owner, shares);

        if (feeShares > 0) _mint(feeRecipient, feeShares);

        adapter.withdraw(assets, receiver, address(this));

        emit Withdraw(msg.sender, receiver, owner, assets, shares);
    688: owner,
                                spender,
                                value,
                                nonces[owner]++,
    702: if (recoveredAddress == address(0) || recoveredAddress != owner)

The same `owner` variable is in the contract `OwnedUpgradeable:`

    10: address public owner;

This use causes compilers to issue warnings, negatively affecting checking and code readability.

### [L-03] INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY

**src\vault\Vault.sol**

The `initialize()` function can be called by anybody when the contract is not initialized.
More importantly, if someone else runs this function, they will have full authority because of the `__Ownable_init()` function. 

    57: function initialize(
        IERC20 asset_,
        IERC4626 adapter_,
        VaultFees calldata fees_,
        address feeRecipient_,
        address owner
    ) external initializer {
        __ERC20_init(
            string.concat(
                "Popcorn ",
                IERC20Metadata(address(asset_)).name(),
                " Vault"
            ),
            string.concat("pop-", IERC20Metadata(address(asset_)).symbol())
        );
        __Owned_init(owner); 

        if (address(asset_) == address(0)) revert InvalidAsset(); 
        if (address(asset_) != adapter_.asset()) revert InvalidAdapter();

        asset = asset_;
        adapter = adapter_;

        asset.approve(address(adapter_), type(uint256).max);

        _decimals = IERC20Metadata(address(asset_)).decimals();

        INITIAL_CHAIN_ID = block.chainid;
        INITIAL_DOMAIN_SEPARATOR = computeDomainSeparator();

        feesUpdatedAt = block.timestamp;
        fees = fees_;

        if (feeRecipient_ == address(0)) revert InvalidFeeRecipient();
        feeRecipient = feeRecipient_;

        contractName = keccak256(
            abi.encodePacked("Popcorn", name(), block.timestamp, "Vault")
        );

        emit VaultInitialized(contractName, address(asset));
    }

Add a control that makes `initialize()` only call the Deployer Contract;

    if (msg.sender != DEPLOYER_ADDRESS) {revert NotDeployer();}


### [L-04] CRITICAL ADDRESS CHANGES SHOULD USE A TWO-STEP PROCEDURE

**src\vault\Vault.sol**

    553: function setFeeRecipient(address _feeRecipient) external onlyOwner {

    594: function changeAdapter() external takeFees

The lack of a two-step procedure for critical operations leaves them error-prone. Consider adding a two-step procedure on the critical functions.
