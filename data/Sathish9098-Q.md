# LOW FINDINGS

##

## [L-1] LACK OF INPUT VALIDATION

For defence-in-depth purpose, it is recommended to perform additional validation against the amount that the user is attempting to deposit, mint, withdraw and redeem to ensure that the submitted amount is valid

File : src/vault/Vault.sol

211 :     function withdraw(
212:        uint256 assets,
213:         address receiver,
214:         address owner
215:      ) public nonReentrant syncFeeCheckpoint returns (uint256 shares) {

+     require(assets>0, "The assets is less");
+     require(assets<= maxWithdraw(owner), "withdraw more than max");

 170 :      function mint(uint256 shares, address receiver)
 171:        public
 172:        nonReentrant
 173:       whenNotPaused
 174:       syncFeeCheckpoint
 175:       returns (uint256 assets)
+            require(shares>0, "The shares is less");
+            require(shares<= maxMint(receiver), "Mint more than max");

134:      function deposit(uint256 assets, address receiver)
135 :        public
136 :        nonReentrant
137 :       whenNotPaused
 138 :      syncFeeCheckpoint
 139:       returns (uint256 shares)

+     require(assets>0, "The assets is less");
+     require(assets<= maxDeposite(receiver), "deposit more than max");

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)

##

## [L-2] INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY

initialize() function can be called by anybody when the contract is not initialized.

Here is a definition of initialize() function

File : src/vault/Vault.sol

[Vault.sol#L57-L72](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L57-L72)

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)

> Recommended Mitigation Steps

Add a control that makes initialize() only call the Deployer Contract or EOA;

if (msg.sender != DEPLOYER_ADDRESS) {
            revert NotDeployer();
        }

##

## [L-3] OWNER CAN RENOUNCE OWNERSHIP

Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The Openzeppelin’s Owned used in this project contract implements renounceOwnership. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner

it is possible to renounce ownership in an "owned" smart contract. The exact process for renouncing ownership will depend on the specific implementation of the "owned" contract, but it typically involves the owner calling a specific function within the contract to transfer ownership to a "null" account, effectively renouncing their ownership of the contract. This can be useful in situations where the current owner no longer wants to be responsible for the contract, or if they need to transfer ownership to another account for security reasons

> onlyOwner functions : 

File : src/vault/Vault.sol

     525:  function proposeFees(VaultFees calldata newFees) external onlyOwner {

    553:   function setFeeRecipient(address _feeRecipient) external onlyOwner {

    578:   function proposeAdapter(IERC4626 newAdapter) external onlyOwner {

    629:   function setQuitPeriod(uint256 _quitPeriod) external onlyOwner {

    643:   function pause() external onlyOwner {

   648 :  function unpause() external onlyOwner {

  [Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)

File:  src/vault/VaultController.sol

      408:    function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

      543:     function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {

      561:    function addTemplateCategories(bytes32[] calldata templateCategories) external onlyOwner {

      723:   function nominateNewAdminProxyOwner(address newOwner) external onlyOwner {

     764:     function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {

     791:    function setHarvestCooldown(uint256 newCooldown) external onlyOwner {

     804:    function setAdapterHarvestCooldowns(address[] calldata adapters) external onlyOwner {

    832:    function setDeploymentController(IDeploymentController _deploymentController) external onlyOwner {

    864:    function setActiveTemplateId(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

  

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol)



> Recommended Mitigation Steps:

We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design



# NON CRITICAL FINDINGS 

##

## [NC -1]  USE A MORE RECENT VERSION OF SOLIDITY . LATEST SOLIDITY VERSION IS 0.8.17 . LATEST VERSION HAVE MORE ADVANCED FEATURES .

File : src/vault/PermissionRegistry.sol

     4 : pragma solidity ^0.8.15;

[Link To Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol)

File : src/vault/CloneRegistry.sol

    4 : pragma solidity ^0.8.15;

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol)

File : src/vault/DeploymentController.sol

     4 : pragma solidity ^0.8.15;

[Link To Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol)

File : src/vault/adapter/yearn/YearnAdapter.sol

     4 : pragma solidity ^0.8.15;

[Link To Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol)


##

## [NC-2] TYPOS

File : src/vault/CloneRegistry.sol

     /// @audit registerd   = >  registered
 
     14 :   * Is used by `VaultController` to check if a target is a registerd clone.

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol)

##

## [NC-3] LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTIONS

To record the init parameters for off-chain monitoring and transparency reasons, please consider emitting an event after the initialize() functions

File : src/vault/adapter/beefy/BeefyAdapter.sol

[BeefyAdapter.sol#L46-L84](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L46-L84)


[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol)

##

## [NC-4]  FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this

[https://docs.soliditylang.org/en/v0.8.17/style-guide.html](https://docs.soliditylang.org/en/v0.8.17/style-guide.html)

Functions should be grouped according to their visibility and ordered:

constructor

receive function (if exists)

fallback function (if exists)

external

public

internal

private

within a grouping, place the view and pure functions last.

##

## [NC -5]  FOR FUNCTIONS, FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS


File : src/vault/Vault.sol

[Vault.sol#L716] (https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L716)


[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)

The above codes don’t follow Solidity’s standard naming convention:

internal and private functions: the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

File : src/utils/MultiRewardStaking.sol

   487:  function DOMAIN_SEPARATOR() public view returns (bytes32) {

   491:  function computeDomainSeparator() internal view virtual returns (bytes32) {

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol)


> Description

The above codes don’t follow Solidity’s standard naming convention,

internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

public and external functions : only mixedCase

constant variable : UPPERCASEWITH_UNDERSCORES (separated by uppercase and underscore)

>

##

## [NC-6]  LINES ARE TOO LONG

Usually lines in source code are limited to 80 characters. Today’s screens are much larger so it’s reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length

File : src/utils/MultiRewardStaking.sol

22:   * Only the owner can add new tokens as rewards. Once added they cant be removed or changed. RewardsSpeed can only be adjusted if the rewardsSpeed is not 0

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol)

##

## [NC-7]  OMISSIONS IN EVENTS

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts.

However, some events are missing important parameters. The events should include the new value and old value where possible:

Events with no old value

File : src/vault/VaultController.sol

    function nominateNewAdminProxyOwner(address newOwner) external onlyOwner {
    adminProxy.nominateNewOwner(newOwner);
  }


   function acceptAdminProxyOwnership() external {
    adminProxy.acceptOwnership();
  }

File: Owned.sol

    function nominateNewOwner(address _owner) external virtual onlyOwner {
    nominatedOwner = _owner;
    emit OwnerNominated(_owner);
    }


    function acceptOwnership() external virtual {
    require(msg.sender == nominatedOwner, "You must be nominated before you can accept ownership");
    emit OwnerChanged(owner, nominatedOwner);
    owner = nominatedOwner;
    nominatedOwner = address(0);
    }


[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol)

##

##  [NC-8]  NO SAME VALUE INPUT CONTROL

  File : src/vault/VaultController.sol

    function nominateNewAdminProxyOwner(address newOwner) external onlyOwner {
    adminProxy.nominateNewOwner(newOwner);
  }


   function acceptAdminProxyOwnership() external {
    adminProxy.acceptOwnership();
  }

[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol)

File: src/utils/Owned.sol

    function nominateNewOwner(address _owner) external virtual onlyOwner {
    nominatedOwner = _owner;
    emit OwnerNominated(_owner);
    }


    function acceptOwnership() external virtual {
    require(msg.sender == nominatedOwner, "You must be nominated before you can accept ownership");
    emit OwnerChanged(owner, nominatedOwner);
    owner = nominatedOwner;
    nominatedOwner = address(0);
    }

[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/Owned.sol)


Recommended Mitigation Steps

Add code like this; if (owner == _owner) revert ADDRESS_SAME();

##

## [NC-9]  Shorter inheritance list


File: src/vault/adapter/abstracts/AdapterBase.sol

    abstract contract AdapterBase is
    ERC4626Upgradeable,
    PausableUpgradeable,
    OwnedUpgradeable,
    ReentrancyGuardUpgradeable,
    EIP165,
    OnlyStrategy

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol)










NC-1	Missing checks for address(0) when assigning values to address state variables	1
NC-2	Return values of approve() not checked	12
NC-3	Event is missing indexed fields	24
NC-4	Functions not used internally could be marked external	15

L-1	abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()	1
L-2	Do not use deprecated library functions	1
L-3	Empty Function Body - Consider commenting why	10
L-4	Initializers could be front-run	14
L-5	Unsafe ERC20 operation(s)	11

