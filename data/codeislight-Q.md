- MultiRewardEscrow.setFees() , checking the tokenFees is better to be inclusive for 10% to make it easier and rounded number as a max value.
```
      if (tokenFees[i] > 1e17) revert DontGetGreedy(tokenFees[i]);

```
instead of 
```
      if (tokenFees[i] >= 1e17) revert DontGetGreedy(tokenFees[i]);

```
- advised to at least use uint40 as a timestamp, since the max value for uint32 would overflow or revert after 83 years.
instances:

- using a single bool endorsed would be sufficient, since there is no special case of canCreate being only acceptable for endoresed or only checking for ones that are not rejected. as at the end they are serving the same purpose. a user can either be rejected or endorsed.

checking for endorsed => endorsed ? yes => pass
checking for endorsed => endorsed ? no => reject
checking for reject => rejected ? yes => reject
checking for reject => rejected ? no => pass

valid if endorsed or not rejected, where the same behavior can be achieved by using endorsed boolean alone.

instead of:
```
if (
      permissionRegistry.endorsed(address(1))
        ? !permissionRegistry.endorsed(msg.sender)
        : permissionRegistry.rejected(msg.sender)
    ) revert NotAllowed(msg.sender);
```
```
  mapping(address => Permission) public permissions;

function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {
    uint256 len = targets.length;
    if (len != newPermissions.length) revert Mismatch();

    for (uint256 i = 0; i < len; i++) {
      if (newPermissions[i].endorsed && newPermissions[i].rejected) revert Mismatch();

      emit PermissionSet(targets[i], newPermissions[i].endorsed, newPermissions[i].rejected);

      permissions[targets[i]] = newPermissions[i];
    }
  }
```
we use:
```
if (!permissionRegistry.endorsed(msg.sender)) revert NotAllowed(msg.sender);
```

```
  mapping(address => bool) public permissions;

function setPermissions(address[] calldata targets, bool[] calldata newPermissions) external onlyOwner {
    uint256 len = targets.length;
    if (len != newPermissions.length) revert Mismatch();

    for (uint256 i = 0; i < len; i++) {

      permissions[targets[i]] = newPermissions[i];

      emit PermissionSet(targets[i], newPermissions[i]);
    }
  }
```
- some functions lacks checking for default values, i.e: TemplateRegistry.addTemplateCategory() lacks checking for bytes32(0)

- TemplateRegistry.addTemplate() lacks checking template.implementation address as a non zero address, or better be checking for the implementation contract code size to be greater than zero.

- TemplateRegistry.toggleTemplateEndorsement() lacks checking that the templateCategory exists in templateCategoryExists mapping.

- no need to use  a modifier if it is used in 1 function, refering to takeFees modifier and takeManagementAndPerformanceFees function. it is advised to move the modifier body to the function.

instance: in Vault : takeManagementAndPerformanceFees and takeFees
instance: in AdapterBase : harvest and takeFees

- prepend internal function names with `_`, i.e instead of computeDomainSeparator(), we may use _computeDomainSeparator()

- VaultController.deployVault(), it needs to check for staking contract code size instead of zero address.

-  AdapterBase.__AdapterBase_init() lacks checking for zero address _owner, and check for code size greater than zero for asset and _strategy.

-  AdapterBase.__AdapterBase_init() lacks checking for _harvestCooldown bound value validity, it is advised to use setHarvestCooldown() or manually check for a 1 day maximum period.

- add an extra check to ensure that in BeefyAdapter.initialize(), _beefyBooster is a contract with a size greater than zero.

- YearnAdapter.initialize() lacks ensuring that (IYearn(yVault) == _asset)