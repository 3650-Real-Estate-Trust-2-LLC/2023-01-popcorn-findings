# Functions that have return parameters don't have the NatSpec  for those return parameters 

It is really useful to have the return parameter as part of the NatSpec. By including the return parameters in the NatSpec, you can ensure that the code is more easily understood and analyzed, which can improve the overall quality and security of the smart contract.

Example: 

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneFactory.sol#L34-L48
 
    @notice Clones an implementation and initializes the clone. Caller must be owner. (DeploymentController)
    @param template The template to use for the deployment. (See TemplateRegistry for more details)
    @param data The data to pass to the clone's initializer.

   **Add**

    @return clone The address of the clone.
