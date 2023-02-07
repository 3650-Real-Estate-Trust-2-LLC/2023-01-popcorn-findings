## Title
`getSubmitter` function returns whole vault struct and not only `vault.creator`

## Links to affected code
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L75

## Vulnerability details

### Tools Used
Manual reviewing

### Recommended Mitigation Steps
```
    function getSubmitter(address vault) external view returns (address) {
        return metadata[vault].creator;
    }
```


## Title
Add function for `feeRecipient` change in `MultiRewardEscrow.sol` contract

## Links to affected code
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L191

## Vulnerability details
### Impact
If account `feeRecipient` would be compromised, or the protocol owner wants from some other reason to change this address, it must be setter function `setFeeRecipient`.

### Tools Used
Manual reviewing

### Recommended Mitigation Steps
```
    function setFeeRecipient(address _feeRecipient) external onlyOwner {
        if (_feeRecipient == address(0)) revert ZeroAddress();
        feeRecipient = _feeRecipient;
    }
```



## Title
Missing check is `template.implementation` different than `address(0)` in function `addTemplate`

## Links to affected code
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L67

## Vulnerability details

### Impact
When the contract owner tries to clone the template where the implementation address is `0`, it will get an error. The additional problem is that there is no template update function, so there is not possible to change the implementation address.

### Tools Used
Manual reviewing

### Recommended Mitigation Steps
```
    function addTemplate(
        bytes32 templateCategory,
        bytes32 templateId,
        Template memory template
    ) external onlyOwner {
        if (!templateCategoryExists[templateCategory]) revert KeyNotFound(templateCategory);
        if (templateExists[templateId]) revert TemplateExists(templateId);
        if (template.implementation == address(0)) revert ZeroAddress();

        template.endorsed = false;
        templates[templateCategory][templateId] = template;

        templateIds[templateCategory].push(templateId);
        templateExists[templateId] = true;

        emit TemplateAdded(templateCategory, templateId, template.implementation);
    }
```



## Title
Unnecessary `_verifyEqualArrayLength`

## Links to affected code
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L544

## Vulnerability details

### Impact
In line [MultiRewardEscrow.sol:208](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L208) there is a same check, so, [VaultController.sol#L544](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L544) is not needed and could be removed

### Tools Used
Manual reviewing

### Recommended Mitigation Steps
Remove line [VaultController.sol#L544](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L544)



## Title
Unnecessary `lastUpdatedTimestamp` update in `fundReward` function

## Links to affected code
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L346

## Vulnerability details

### Impact
In line [MultiRewardStaking.sol#L339](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L339) is called function `_accrueRewards` where there is a same update [MultiRewardStaking.sol#L409](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L409)

### Tools Used
Manual reviewing

### Recommended Mitigation Steps
Remove line [MultiRewardStaking.sol#L346](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L346)



## Title
Function state mutability can be restricted to view

## Links to affected code
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L351
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L242
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L667



## Title
Unused local variable

## Links to affected code
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L448



## Title
Unused function parameter

## Links to affected code
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L390



## Title
Return value of low-level calls not used

## Links to affected code
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L444



## Title
Declaration shadows an existing declaration

## Links to affected code
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L388
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L124
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L446
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L60
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L176
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L196
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L213
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L633
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L62
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L214
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L256
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L665

## Vulnerability details

### Recommended Mitigation Steps
Rename variables from code lines from section `Links to affected code`
