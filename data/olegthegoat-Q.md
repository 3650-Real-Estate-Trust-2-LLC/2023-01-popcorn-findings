# QA Report
One issue was found with low impact.

# L01 - setPermissions function not checking for valid boolean

On line 43 in PermissionRegistry.sol, the only check that exists is making sure that the user doesn't have both permissions set to true for endorsed and rejected. The code is not taking into account that it is possible to pass down other value types that are NOT boolean. If endorsed and rejected are set to another value that is not true or false, it could result in unexpected behavior.

## Impact:
LOW

## Tool used:
Manual Analysis

## Mitigation:
- In PermissionRegistry.sol, before Line 43 inside the for loop, add validation to see if both rejected and endorsed are valid booleans.  

Affected Code:
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L43