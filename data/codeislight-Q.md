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

- no need to directly cast function array input length to uint8, since if the length is greater than 255, it would overflow without any actual damage to the logic execution, recommended to instead assign it to uint256 variable directly without any down casting.

as an elaboration, you may try the following input on the following function:
input: [5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,6,5,5,5,5]

```
contract Test {
    function test(uint256[] calldata array) public returns(uint8){
        return uint8(array.length);
    }
}
```
it would return 1.
