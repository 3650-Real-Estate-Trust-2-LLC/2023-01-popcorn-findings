# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1 | Existing clone can be added multiple times in `CloneRegistry` | Low | 1 |
| 2 | Vault fees can be set greater than `1e18` in the `initialize` function | Low | 1 |
| 3 | `harvestCooldown` value not verified in the `__AdapterBase_init` function | Low | 1 |
| 4 | `feeRecipient` can be set to `address(0)` | Low | 1 |
| 5 | The function `getSubmitter` should return the vault creator address | NC | 1 |
| 6 | Duplicated input check statements should be refactored to a modifier  | NC |  4 |
| 7 | `constant` should be used instead of `immutable` | NC |  4 |


## Findings

### 1- Existing clone can be added multiple times in `CloneRegistry` :

The `addClone` function from the `CloneRegistry` contract does not check if the input clone address already exists or not and it proceeds immediatly to adding the clone address to the `clones` and `allClones` arrays thus creating duplicates addresses inside them, in the current protocol implementation this does not cause any issue as only the `DeploymentController` is allowed to call the `addClone` function which actually ensures that only unique clones are added but if the contracts get upgraded in the future and the function caller (other parties get allowed to call the function) changes this may create a surface of attack or could lead to wrong protocol behaviour.

#### Risk : Low

#### Proof of Concept

The issue occurs in the `addClone` below :

File: vault/CloneRegistry.sol [Line 41-51](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneRegistry.sol#L41-L51)
```
function addClone(
    bytes32 templateCategory,
    bytes32 templateId,
    address clone
) external onlyOwner {
    cloneExists[clone] = true;
    clones[templateCategory][templateId].push(clone);
    allClones.push(clone);

    emit CloneAdded(clone);
}
```

The function does not check if the clone already exists meaning that `cloneExists[clone] == true` and thus can add the same clone multiple times.

#### Mitigation

Add a clone existance check in the `addClone` function :

```
function addClone(
    bytes32 templateCategory,
    bytes32 templateId,
    address clone
) external onlyOwner {
    if (cloneExists[clone]) revert AlreadyExists(clone);

    cloneExists[clone] = true;
    clones[templateCategory][templateId].push(clone);
    allClones.push(clone);

    emit CloneAdded(clone);
}
```

### 2- Vault fees can be set greater than `1e18` in the `initialize` function :

The Vaut contract implements 4 types of fees (deposit, withdrawal, management, performance) collected when the user deposits or withdraw tokens, those fees are calculated in basis point (BPS) where 1e18 is equivalent to 100% of the given amount.

The function `proposeFees` is used to set the value for new fees and it ensures that their respective values are always less than 1e18, but when the vault is initialized during deployment the `initialize` function (of the Vault contract) does not check the value of the fees set and thus it allows fees that are greater than 1e18.

This does not have a big impact as users can't call the `deposit` or `mint` functions (as they both underflow in this case) but this will block the vault until the owner proposes new fees and then you need to wait until the `quitPeriod` period has passed to update the fees and open the vault.

#### Risk : Low

#### Proof of Concept

The issue occurs in the `initialize` which does not contain any check for the value of `fees` :

File: vault/Vault.sol [Line 57-98](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L57-L98)
```
function initialize(
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
```

As you can see the `fees` are set immediately without any check on their values.

#### Mitigation
Add a check in the `initialize` function of the Vault contract to ensure that the `fees` are always less than 1e18.

### 3- `harvestCooldown` value not verified in the `__AdapterBase_init` function :

The `harvestCooldown` variable is used to determine the time delay between two harvest operation and by the protocol specifications it is supposed to always be less than 1 day and this is ensured in the function `setHarvestCooldown`, but when the adapter contract is initially deployed the initializer function `__AdapterBase_init` does not verify that the value of `harvestCooldown` provided is in fact less than 1 day and when deployed the value of `harvestCooldown` can have any value from 0 to 2^256(uint256). 

This issue has low impact on the protocol as the owner can immediatly call `setHarvestCooldown` to update the value but it can still cause problems if the error is not noticed at first.

#### Risk : Low

#### Proof of Concept

The issue occurs in the `__AdapterBase_init` which does not contain any check for the value of `harvestCooldown` :

File: vault/adapter/abstracts/AdapterBase.sol [Line 55-87](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L55-L87)
```
function __AdapterBase_init(bytes memory popERC4626InitData)
    internal
    onlyInitializing
{
    (
        address asset,
        address _owner,
        address _strategy,
        uint256 _harvestCooldown,
        bytes4[8] memory _requiredSigs,
        bytes memory _strategyConfig
    ) = abi.decode(
            popERC4626InitData,
            (address, address, address, uint256, bytes4[8], bytes)
        );
    __Owned_init(_owner);
    __Pausable_init();
    __ERC4626_init(IERC20Metadata(asset));

    INITIAL_CHAIN_ID = block.chainid;
    INITIAL_DOMAIN_SEPARATOR = computeDomainSeparator();

    _decimals = IERC20Metadata(asset).decimals();

    strategy = IStrategy(_strategy);
    strategyConfig = _strategyConfig;
    harvestCooldown = _harvestCooldown;

    if (_strategy != address(0)) _verifyAndSetupStrategy(_requiredSigs);

    highWaterMark = 1e18;
    lastHarvest = block.timestamp;
}
```

#### Mitigation

Add a check in the `__AdapterBase_init` function to ensure that `harvestCooldown < 1 days` or call the function `setHarvestCooldown` directly inside the `__AdapterBase_init` function.

### 4- `feeRecipient` can be set to `address(0)` :

The address of `feeRecipient` in the MultiRewardEscrow contract can be set by accident to `address(0)` in the constructor.

#### Risk : Low

#### Proof of Concept
Instances include:

File: utils/MultiRewardEscrow.sol [Line 31](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L31)
```
feeRecipient = _feeRecipient;
```

#### Mitigation
Add non-zero address checks in the constructor of the MultiRewardEscrow contract.


### 5- The function `getSubmitter` should return the vault creator address :

In the `VaultRegistry` contract the function `getSubmitter` is supposed to return the address of the vault creator when it is called but instead it returns the whole vault struct which doesn't make sense as there is already the `getVault` function which is responsible for that, this issue is not really critical for the protocol as the function is marked as view and is not used in other contracts but this error could cause problems for others parties who tries to integrate with the protocol as the `getSubmitter` function put in the `IVaultRegistry` interface actually returns an address.

#### Risk : Non Critical

#### Proof of Concept

Issue occurs in the instance below :

File: vault/VaultRegistry.sol [Line 75-77](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L75-L77)
```
/** @audit
    should return metadata[vault].creator
*/
function getSubmitter(address vault) external view returns (VaultMetadata memory) {
    return metadata[vault];
}
```

And the `IVaultRegistry` interface also confirms that the function should return an address :

File: interfaces/vault/IVaultRegistry.sol [Line 27](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVaultRegistry.sol#L27)
```
function getSubmitter(address vault) external view returns (address);
```

#### Mitigation
Return the address of the vault creator `metadata[vault].creator` in the `getSubmitter` function.

### 6- Duplicated input check statements should be refactored to a modifier :

Input check statements used multiple times inside a contract should be refactored to a modifier for better readability and also to save gas.

#### Risk : Non Critical

#### Proof of Concept

There are 4 instances of this issue :

```
File: vault/Vault.sol

141     if (receiver == address(0)) revert InvalidReceiver();
177     if (receiver == address(0)) revert InvalidReceiver();
216     if (receiver == address(0)) revert InvalidReceiver();
258     if (receiver == address(0)) revert InvalidReceiver();
```

Those checks should be replaced by a `ValidReceiver` modifier as follow :

```
modifier ValidReceiver(address receiver){
    if (receiver == address(0)) revert InvalidReceiver();
    _;
}
```

### 7- `constant` should be used instead of `immutable` :

The type `constant` should be used for variables that have values set immediately in the contract code and the `immutable` type should be used for variables declared in the constructor.

#### Risk : Non Critical

#### Proof of Concept

There are 4 instances of this issue :

File: vault/VaultController.sol [Line 36-40](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L36-L40)
```
bytes32 public immutable VAULT = "Vault";
bytes32 public immutable ADAPTER = "Adapter";
bytes32 public immutable STRATEGY = "Strategy";
bytes32 public immutable STAKING = "Staking";
bytes4 internal immutable DEPLOY_SIG = bytes4(keccak256("deploy(bytes32,bytes32,bytes)"));
```

#### Mitigation
Use `constant` instead of `immutable` in the instances aforementioned.