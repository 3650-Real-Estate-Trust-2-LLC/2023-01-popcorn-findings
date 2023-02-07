Overcomplicated logic in TemplateRegistry.sol for interactions with templates.

Issue:
To use templates it's required to always use categories. But since there can't be multiple templates with same templateIds in different categories (https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L73) - all interactions with templates could be achived by calling them by templateId only. 
By the look of code base and logic of the project (discussed it with Popcorn developer for a little) - categories are only needed to add grouping possibility for templates and there is no need to always check for template category while template is called. 
getTemplate method (https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L123) is used at 4 different places and maybe will be used somewhere else in future.
In my opinion storing and calling of templates could be simplified and categories might be used for grouping only and used in separate checks when it's needed. 

Possilbe solution:
For example, instead of "TemplateCategory => TemplateId => Template" mapping there could be two separate mappings: "TemplateId => Template" and "TemplateCategory => TemplateId". This way it would be easier to interact with template and it might help make code easier to update in future if more logic will be added to Categories and Templates. 