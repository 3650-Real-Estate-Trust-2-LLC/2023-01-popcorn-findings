## Low Risk Findings
1. Category check is needed when toggling template endorsement, In TemplateRegistry.sol - Line 102. Template category is not checked if it exists or not. And due to this a small mistake toggle some other nonExisting template
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L102

Here is a test to demonstrate it:
```function test__toggleTemplateEndorsement_wrong_Category() public {
    bytes32 templateCategory1 = "templateCategory1";
    ClonableWithInitData clonableWithInitData = new ClonableWithInitData();
    registry.addTemplateCategory(templateCategory);
    registry.addTemplate(
      templateCategory,
      templateId,
      Template({
        implementation: address(clonableWithInitData),
        endorsed: false,
        metadataCid: metadataCid,
        requiresInitData: true,
        registry: address(0x2222),
        requiredSigs: reqSigs
      })
    );

    vm.expectEmit(true, true, true, false, address(registry));
    emit TemplateEndorsementToggled(templateCategory, templateId, false, true);

    registry.toggleTemplateEndorsement(templateCategory1, templateId);

    Template memory template = registry.getTemplate(templateCategory, templateId);
    assertFalse(template.endorsed);
    template = registry.getTemplate(templateCategory1, templateId);
    assertTrue(template.endorsed);
  }
```

2. Missing call to __ReentrancyGuard_init() in Vault.sol - https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L57

3. Missing call to __Pausable_init() in Vault.sol - https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L57

4. Function _totalAssets shadows an existing declaration L258 and L388 - https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L258 and https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L388

5. changeFees() can be called by anyone because it does not have the onlyOwner modifier - https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L540

6. Modifiers are being used to make state changes and call other functions 
 a. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L480
 b. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L496
 c. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L371
 d. https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L559

## Non-Critical Findings
1. Unused variable vault in _registerVault (L390)
2. Code readability is poor. Consider writing the code in order with initializing and declaring all state variables first and then writing the function logic.