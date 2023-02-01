Missing input validation vulnerability 

Impact:
the function bellow "setPermissions()" does not have proper input validation checks as a result the caller of this function is able to pass an array with different lengths for the "targets" and "newPermissions" parameters, a mismatch could lead to an exception or undefined behavior

Proof of Concept: 

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L38-L48

when calling the 'setPermissions' function with an array of target addresses and an array of permissions that are not of equal length it reverts with a 'Mismatch' error indicating that the input validation check has failed

modifying the 'targets' array to be longer than the length of the 'newPermissions' array then Calling the 'setPermissions' function again it continues executing instead of reverting with a 'Mismatch' error indicating that the input validation check has been bypassed.

tools used: 
Visual Studio Code, Remix

Recommended Mitigation:
Add input validation to check that the arrays lengths match :

require(targets.length == newPermissions.length, "Mismatch");

this will ensure that the "setPermissions()" function is only executed when the arrays are of equal lengths and will prevent any potential exploitation




