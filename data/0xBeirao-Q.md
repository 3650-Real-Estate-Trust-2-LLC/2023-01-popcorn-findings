### **StrategyBase** contract must be **abstract**

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/strategy/StrategyBase.sol#L9

The **StrategyBase** contract can be restricted to **abstract**.

### Lack of documentation for adapter, strategy and vault creator

There is no "user manual" type of documentation for vault creators.  
Popcorn's goal is to make it easy and safe for anyone to create a vault, but we actually need to dive deep into the code base to understand how to create one. This kind of documentation needs to exist.

### No check for **_fees** in the **Vault** constructor

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L88

Just check that the initial fees are correct by implementing this logic in the constructor of **Vault** :
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L526-L531


### Check for mandatory parameters in **VaultMetaData** and **Template** struct

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L44-L53

``` js
struct VaultMetadata {
  /// @notice Vault address
  address vault;
  /// @notice Staking contract for the vault
  address staking;
  /// @notice Owner and Vault creator
  address creator;
  /// @notice IPFS CID of vault metadata
  string metadataCID;
  /// @notice OPTIONAL - If the asset is an Lp Token these are its underlying assets
  address[8] swapTokenAddresses;
  /// @notice OPTIONAL - If the asset is an Lp Token its the pool address
  address swapAddress;
  /// @notice OPTIONAL - If the asset is an Lp Token this is the identifier of the exchange (1 = curve)
  uint256 exchange;
}
```
According to the natspecs above (in **IVaultRegistry**), if **swapTokenAddresses**,**swapAddress** and **exchange** are OPTIONAL then **vault**, **staking**, **creator** and **metadataCID** should be mandatory ? If it's the case, the **registerVault()** function should ckeck if they are not null.

Just add something like that to **registerVault()** to check : 
``` js
if (_metadata.vault == address(0) 
      || _metadata.staking == address(0) 
      || _metadata.creator == address(0) 
      || _metadata.metadataCID.length == 0) revert();
```
There is no functions to delete a vault is registry. Must not push incomplete vault structure.


Same for **addTemplate()**: https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L67-L82

Some parameters of the **Template** structure are mandatory :

``` js 
/// @notice Template used for creating new clones
struct Template {
  /// @Notice Cloneable implementation address
  address implementation;
  /// @Notice implementations can only be cloned if endorsed
  bool endorsed;
  /// @Notice Optional - Metadata CID which can be used by the frontend to add informations to a vault/adapter...
  string metadataCid;
  /// @Notice If true, the implementation will require an init data to be passed to the clone function
  bool requiresInitData;
  /// @Notice Optional - Address of an registry which can be used in an adapter initialization
  address registry;
  /// @Notice Optional - Only used by Strategies. EIP-165 Signatures of an adapter required by a strategy
  bytes4[8] requiredSigs;
}
```
And parameters check is needed in **addTemplate()**.


### Consider check all parameters in **toggleTemplateEndorsement()**

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L102-L109

Consider checking both **templateCategory** and **templateId** existence in the registry.

from : 
``` js 
function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {
if (!templateExists[templateId]) revert KeyNotFound(templateId);

bool oldEndorsement = templates[templateCategory][templateId].endorsed;
templates[templateCategory][templateId].endorsed = !oldEndorsement;

emit TemplateEndorsementToggled(templateCategory, templateId, oldEndorsement, !oldEndorsement);
}
```

to : 
``` js 
function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {
    if (!templateExists[templateId]) revert KeyNotFound(templateId);
    if (!templateCategoryExists[templateCategory]) revert KeyNotFound(templateCategory);
    
    bool oldEndorsement = templates[templateCategory][templateId].endorsed;
    templates[templateCategory][templateId].endorsed = !oldEndorsement;

    emit TemplateEndorsementToggled(templateCategory, templateId, oldEndorsement, !oldEndorsement);
}
```

### Wrong natspecs for **CloneRegistry**, **CloneFactory** and **TemplateRegistry**

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneRegistry.sol#L21
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneFactory.sol#L22
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L23


According to the documentation, the direct **_owner** of **CloneRegistry.sol**, **CloneFactory.sol** and **TemplateRegistry.sol** should be **DeploymentController.sol** not **AdminProxy**

### Wrong natspecs for **addTemplate()**

Everyone can call **addTemplate()**. 

from : 
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/DeploymentController.sol#L60

to :
```
@notice Every user can adds a new template. 
```

### PermissionRegistry can be simplified 

This endorsed/rejected system was put in place to switch frrom a **whitelist** to a **blacklist** access when Popcorn will feel confortable by open up the protocol to every one. 
In reality, if someone got he's Template rejected, he can easily create a new one and pursue he's malicious behaviour. So I don't think the **blacklist** access mode is relevant.
This contract can therefore be simplified by replacing the permission structure with a simple Boolean variable: endorsed.

By making this modification changes need to be done on **VaultController** by changing these lines :  
 - https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L682-L684
 - https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L706-L708
 - https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IPermissionRegistry.sol (just rewiew the overall adapter)

Moreover, to save gas, the **i++** can be become **unchecked { ++i; }** and be placed at the end of the for loop and test if there is a length in both parameters. 

However I don't see any vulnerabilities but complexity is more likely to lead to security issues.

from : 
``` js
pragma solidity ^0.8.15;

import { Owned } from "../utils/Owned.sol";
import { Permission } from "../interfaces/vault/IPermissionRegistry.sol";

contract PermissionRegistry is Owned {
  constructor(address _owner) Owned(_owner) {}
  mapping(address => Permission) public permissions;
  event PermissionSet(address target, bool newEndorsement, bool newRejection);
  error Mismatch();

  function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {
    uint256 len = targets.length;
    if (len != newPermissions.length) revert Mismatch();

    for (uint256 i = 0; i < len; i++) {
      if (newPermissions[i].endorsed && newPermissions[i].rejected) revert Mismatch();

      emit PermissionSet(targets[i], newPermissions[i].endorsed, newPermissions[i].rejected);

      permissions[targets[i]] = newPermissions[i];
    }
  }

  function endorsed(address target) external view returns (bool) {
    return permissions[target].endorsed;
  }

  function rejected(address target) external view returns (bool) {
    return permissions[target].rejected;
  }
}
```

to : 
``` js
pragma solidity ^0.8.15;

import { Owned } from "../utils/Owned.sol";

contract PermissionRegistry is Owned {
  constructor(address _owner) Owned(_owner) {}
  mapping(address => bool) public permissions;
  event PermissionSet(address target, bool newEndorsement, bool newRejection);

  error EmptyParams();
  error Mismatch();

  function setPermissions(address[] calldata targets, bool[] calldata newPermissions) external onlyOwner {
    uint256 len = targets.length;
    if (len == 0 || newPermissions.length == 0) revert EmptyParams();
    if (len != newPermissions.length) revert Mismatch();

    for (uint256 i = 0; i < len; ) {

      emit PermissionSet(targets[i], newPermissions[i].endorsed, newPermissions[i].rejected);

      permissions[targets[i]] = newPermissions[i];

      unchecked { ++i; }
    }
  }

  function endorsed(address target) external view returns (bool) {
    return permissions[target].endorsed;
  }
}
```

### testFail__initialize_adapter_addressZero() wrong test (no in scope but just just a heads up) 

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/test/vault/Vault.t.sol#L159

Should be :
```solidity 
IERC4626(address(0)),
```

### In Vault.sol withdraw() function should check if the amount pass in (*assets*) is not null 

Just to revert the transaction if the "assets == 0". 

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L211-L240


### **harvest()** cooldown fonctionality do not work properly

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L438-L450

The **lastHarvest** variable used to limit **harvest()** calls is never reassigned. This means that the cooldown mechanism only works the first time **harvest()** is called. 
This features was added to let a strategy compound interest before harvesting. This way, earns will always be greater than gas price needed to harvest. 

This must be added to the **harvest()** function :
```js 
lastHarvest = block.timestamp();
```

Maybe a medium because it's a core functionality but the security issue of it isn't high.