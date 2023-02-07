# [01-I] Already existing clone can be added in CloneRegistry
Currently, only the owner can add new clone in CloneRegistry but to prevent from some mistakes from adding of existing clone should be good to have check if the clone is already added.
File CloneRegistry.sol: line [41](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L41)

# [02-I] Mismatched comment above external function
Mismatched comment with function logic.
DeploymentController.sol: [60][https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L60]

# [03-I] Template with invalid implementation address can be added
Zero address or just EOA address can be setted to implementation of the Template. Nowhere is checked if the implementation address is not zero or is a smart contract.
TemplateRegistry.sol: (https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L67-L81)

# [04-I] acceptDependencyOwnership can be called by everyone
acceptDependencyOwnership should be called only from the AdminProxy which will expect to accept ownership. 
File DeploymentController.sol: [131](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L131)

# [05-I] Immutable variable not setted in constructor should be mark as constant
State variables can be declared as constant or immutable. In both cases, the variables cannot be modified after the contract has been constructed. For constant variables, the value has to be fixed at compile-time, while for immutable, it can still be assigned at construction time.

VaultController.sol: [36-40](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L36-L40)

# [08-I] Missed NatSpec
In some places in the contract is missed good NatSpec comment. The  good commented code increase readability.

# [09-I] Missed check for MALLEABILITY value for S
In function permit before ecrecover() is missed check for malleable value for S.
resource: https://eips.ethereum.org/EIPS/eip-1271 File MultiRewardStaking: [445-L485](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L445-L485) Also in [AdapterBase](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L667)

# [10-I] Do not use magic varaible
Do not use magic variable use declare constant
AdapterBase.sol: [502](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L502)
Use constant variable for version for computing of domain separator. 
File Vault.sol: line [724] (https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L724)
