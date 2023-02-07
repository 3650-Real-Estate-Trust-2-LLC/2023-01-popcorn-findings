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

> Recommended Mitigation Steps : 

     require(assets>0, "The assets is less");
     require(assets<= maxWithdraw(owner), "withdraw more than max");

 170 :      function mint(uint256 shares, address receiver)
 171:        public
 172:        nonReentrant
 173:       whenNotPaused
 174:       syncFeeCheckpoint
 175:       returns (uint256 assets)

> Recommended Mitigation Steps : 

            require(shares>0, "The shares is less");
            require(shares<= maxMint(receiver), "Mint more than max");

134:      function deposit(uint256 assets, address receiver)
135 :        public
136 :        nonReentrant
137 :       whenNotPaused
 138 :      syncFeeCheckpoint
 139:       returns (uint256 shares)

> Recommended Mitigation Steps : 

     require(assets>0, "The assets is less");
     require(assets<= maxDeposite(receiver), "deposit more than max");

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)

##

## [L-2] OWNER CAN RENOUNCE OWNERSHIP

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

##

## [L-3]  DECIMALS() NOT PART OF ERC20 STANDARD

decimals() is not part of the official ERC20 standard and might fail for tokens that do not implement it. While in practice it is very unlikely, as usually most of the tokens implement it, this should still be considered as a potential issue

File : src/vault/Vault.sol

    82:  _decimals = IERC20Metadata(address(asset_)).decimals();

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)

##

## [L-4]  AVOID DUPLICATE CODE

File:  src/vault/VaultController.sol

[pauseAdapters](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L605-L615)

[pauseVaults ](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L618-L628)

The pauseAdapters and pauseVaults methods of the VaultController contract are very similar, the only difference being the following peace of code

>  pauseAdapters()

      610 :   IVault(vaults[i]).adapter(),

>   pauseVaults 

       623 :    vaults[i],

  > Similar to unpauseAdapters and unpauseVaults methods of the VaultController contract are very similar, the only difference being the following peace of code

>  unpauseAdapters ()

     636:    IVault(vaults[i]).adapter(),

>   unpauseVaults ()

     649:   vaults[i],

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol)

>  Recommended Mitigation Steps : 

It’s recommended to reuse the code in order to be more readable and light

##

## [L-5]  NatSpec comments should be increased in contracts

##

## [L-6]  USING VULNERABLE DEPENDENCY OF OPENZEPPELIN. Use Latest non vulnerable version

The package.json configuration file says that the project is using 4.7.3 of OZ which has a not last update version

File: package.json

   26:   "openzeppelin-upgradeable": "npm:@openzeppelin/contracts-upgradeable@4.7.1",


Recommended Mitigation Steps
Use patched versions.

Latest non vulnerable version 4.8.0.

##
## [L-7]  AVOID HARDCODED VALUES

File : src/vault/Vault.sol

   630 :  if (_quitPeriod < 1 days || _quitPeriod > 7 days)  //@audit 1days and 7 days hardcoded values 

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)

File:  src/vault/VaultController.sol

   753:  if (newFee > 2e17) revert InvalidPerformanceFee(newFee);  //@audit 2e17 hardcoded value 

   793:   if (newCooldown > 1 days) revert InvalidHarvestCooldown(newCooldown);  //@audit 1day hardcoded value 

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol)

File:  src/vault/adapter/abstracts/AdapterBase.sol

   502:  if (newCooldown >= 1 days) revert InvalidHarvestCooldown(newCooldown);   //@audit 1day hardcoded value 

   551 :   if (newFee > 2e17) revert InvalidPerformanceFee(newFee);  //@audit 2e17 hardcoded value 

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol)

##

# NON CRITICAL FINDINGS 

##

## [NC -1]  USE A MORE RECENT VERSION OF SOLIDITY . LATEST SOLIDITY VERSION IS 0.8.17 . LATEST VERSION HAVE MORE ADVANCED FEATURES .

> INSTANCES (16)

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

> INSTANCES (4)

File : src/vault/CloneRegistry.sol

     /// @audit registerd   = >  registered
 
     14 :   * Is used by `VaultController` to check if a target is a registerd clone.

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol)

File : src/vault/VaultController.sol

      /// @audit ownes = >  owns

     47:  * @param _adminProxy `AdminProxy` ownes contracts in the vault ecosystem.

     /// @audit informations= >  information's

     85:   * @param metadata Vault metadata for the `VaultRegistry` (Will be used by the frontend for additional informations)
 
     /// @audit auxiliery = >  auxiliary 

    87:  * @dev This function is the one stop solution to create a new vault with all necessary admin functions or auxiliery contracts.
    
[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol)


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

[Vault.sol#L716](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L716)

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
    adminProxy.nominateNewOwner(newOwner);  //@audit nominateNewOwner() Call
     }

##

    function acceptAdminProxyOwnership() external {
    adminProxy.acceptOwnership();   //@audit acceptOwnership() Call
    } 

[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol)
##

File: src/utils/Owned.sol

    function nominateNewOwner(address _owner) external virtual onlyOwner {
    nominatedOwner = _owner;
    emit OwnerNominated(_owner);
     }
##

    function acceptOwnership() external virtual {
    require(msg.sender == nominatedOwner, "You must be nominated before you can accept ownership");
    emit OwnerChanged(owner, nominatedOwner);
    owner = nominatedOwner;
    nominatedOwner = address(0);
    }

[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/Owned.sol)


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

##

## [NC-10]  PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD

##

## [NC-11]  NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES

All interfaces using the floating solidity version instead of fixed version. Interfaces must use fixed version compiler 


> Instances (18)


File:  src/interfaces/IEIP165.sol

2 :  pragma solidity ^0.8.15;


[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IEIP165.sol)

##

## [NC-12]  EMPTY BLOCKS SHOULD BE REMOVED OR EMIT SOMETHING

Description
Code contains empty block

      File: src/utils/EIP165.sol

      7:   function supportsInterface(bytes4 interfaceId) public view virtual returns (bool) {}


      File: src/vault/AdminProxy.sol

      16:   constructor(address _owner) Owned(_owner) {}


      File: src/vault/CloneFactory.sol

      23:   constructor(address _owner) Owned(_owner) {}


      File: src/vault/CloneRegistry.sol

      22:   constructor(address _owner) Owned(_owner) {}


     File: src/vault/PermissionRegistry.sol

     20:   constructor(address _owner) Owned(_owner) {}


     File: src/vault/TemplateRegistry.sol

     24:   constructor(address _owner) Owned(_owner) {}


     File: src/vault/Vault.sol

     477:     {}


     File: src/vault/VaultRegistry.sol

     21:   constructor(address _owner) Owned(_owner) {}


     File: src/vault/adapter/abstracts/WithRewards.sol

     13:   function rewardTokens() external view virtual returns (address[] memory) {}

     15:   function claim() public virtual onlyStrategy {}

##

## [NC-13]  GENERATE PERFECT CODE HEADERS EVERY TIME

Description
I recommend using header for Solidity code layout and readability

[Code Headers](https://github.com/transmissions11/headers)

##

## [NC-14]  NATSPEC IS MISSING

Description
NatSpec is missing for the following functions , constructor and modifier,Events

File : src/vault/Vault.sol

    124:   function deposit(uint256 assets) public returns (uint256) {

    160 :   function mint(uint256 shares) external returns (uint256) {

    200 :   function withdraw(uint256 assets) public returns (uint256) {

    709:    function DOMAIN_SEPARATOR() public view returns (bytes32) {

    108:  event Deposit(

    114:   event Withdraw(


     modifier syncFeeCheckpoint() {
        _;
        highWaterMark = convertToAssets(1e18);
      }

    512:  event NewFeesProposed(VaultFees newFees, uint256 timestamp);

    513:   event ChangedFees(VaultFees oldFees, VaultFees newFees);

    514:   event FeeRecipientUpdated(address oldFeeRecipient, address newFeeRecipient);

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)

##

## [NC-15]  INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS

 [Natspec-Format](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html)

If Return parameters are declared, you must prefix them with ”/// @return”.

Some code analysis programs do analysis by reading NatSpec details, if they can’t see the “@return” tag, they do incomplete analysis.

Recommended Mitigation Steps :
Include return parameters in NatSpec comments

File : src/vault/VaultController.sol

[deployVault()](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L78-L97)

  
[deployStaking() ](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L273-L278)

[Link to Code](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol)


Recommendation Code Style:

		/// @notice information about what a function does
		/// @param pageId The id of the page to get the URI for.
		/// @return Returns a page's URI if it has been minted 
		function tokenURI(uint256 pageId) public view virtual override returns (string memory) {

##

## [NC-16]  AVOID SHADOWING INHERITED STATE VARIABLES

there is a local variable named underlyingBalance_ , but there is a function named _underlyingBalance() with the same name. This use causes compilers to issue warnings, negatively affecting checking and code readability.

File:  src/vault/adapter/abstracts/AdapterBase.sol

222:  uint256 underlyingBalance_ = _underlyingBalance();  

[Link to Code](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol)
			









