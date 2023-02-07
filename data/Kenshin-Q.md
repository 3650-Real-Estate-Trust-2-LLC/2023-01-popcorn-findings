# `feeRecipient` should be marked as immutable
## Permalinks
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L191

## Description
Any unmodifiable state variables should be marked as `immutable`.

## Remediation
Declare the `feeRecipient` as `immutable` variable.

---

# Missing `onlyOwner` modifier for `addTemplate()` function
## Permalinks
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/DeploymentController.sol#L66

## Description
The function `addTemplate()` is supposed to be restricted to owner only as the other functions in the contract. But it is appeared that the `onlyOwner` is missing.

## Remediation
Apply `onlyOwner` modifier to the function.

---

# Missing check for template category existence
## Permalinks
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L102-L109

## Description
Set endorsement of uninitialize is possible because the function does not verify the existence of the template category. Therefore, modify `DeploymentController.deploy()` to [pass the endorsement check](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/DeploymentController.sol#L106) and deploy a new clone to a category that does not exist.

## Remediation
Add the template category existence check to the function.

---