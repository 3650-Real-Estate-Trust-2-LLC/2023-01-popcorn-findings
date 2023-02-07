## Summary

Low Risk Issues List

| Number | Issues Details |
|---|---|
| [L-01] | Direct usage of ecrecover allows signature malleability |
| [L-02] | Missing calls to `__ReentrancyGuard_init` functions of inherited contracts |
| [L-03] | `initialize()` function can be called by anybody |
| [L-04] | A single point of failure |

## [L-01] Direct usage of ecrecover allows signature malleability

### **Impact**

The `permit` function on several occasions calls the Solidity ecrecover function directly to verify the given signature. However, the ecrecover EVM opcode allows for malleable (non-unique) signatures and thus is susceptible to replay attacks. Although a replay attack on this contract is not possible since each user's nonce is used only once, rejecting malleable signatures is considered a best practice.

### **Proof of Concept**
**Referenced code:**
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L664-L672

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L632-L640

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L445-L453

[SWC-117: Signature Malleability](https://swcregistry.io/docs/SWC-117)

[SWC-121: Missing Protection against Signature Replay Attacks](https://swcregistry.io/docs/SWC-121)

**Recommended Mitigation Steps**
Use the `recover` function from [OpenZeppelin's ECDSA](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol) library for signature verification.

## [L-02] Missing calls to `__ReentrancyGuard_init` functions of inherited contracts

Most contracts use the delegateCall proxy pattern and hence their implementations require the use of `initialize()` functions instead of constructors. This requires derived contracts to call the corresponding init functions of their inherited base contracts. This is done in most places except a few.

### **Impact**
The inherited base classes do not get initialized which may lead to undefined behavior.

Missing call to `__ReentrancyGuard_init`:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L64-L72

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L70-L72

### **Recommended Mitigation Steps**
Add missing calls to init functions of inherited contracts.

        __Owned_init(_owner);
        __Pausable_init();
        __ERC4626_init(IERC20Metadata(asset));
    + __ReentrancyGuard_init();

## [L-03] `initialize()` function can be called by anybody

`initialize()` function can be called anybody when the contract is not initialized.

More importantly, if someone else runs this function, they will have full authority because of the `__Ownable_init()` function. Also, there is no 0 address check in the address arguments of the `initialize()` function, which must be defined.

### **Proof of Concept**

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L57-L98

### **Recommended Mitigation Steps**

Add a control that makes `initialize()` only call the Deployer Contract;

    if (msg.sender != DEPLOYER_ADDRESS) {
						revert NotDeployer();
				}


## [L-04] A single point of failure

    7 results - 4 files

    src/utils/MultiRewardStaking.sol:
    10: import { OwnedUpgradeable } from "./OwnedUpgradeable.sol";

    26: contract MultiRewardStaking is ERC4626Upgradeable, OwnedUpgradeable {

    src/utils/OwnedUpgradeable.sol:
    9: contract OwnedUpgradeable is Initializable {
    10    address public owner;

    src/vault/Vault.sol:
    13: import {OwnedUpgradeable} from "../utils/OwnedUpgradeable.sol";
    30:     OwnedUpgradeable
    31  {

    src/vault/adapter/abstracts/AdapterBase.sol:
    15: import {OwnedUpgradeable} from "../../../utils/OwnedUpgradeable.sol";
    30:     OwnedUpgradeable,

### **Impact**

The `owner` role has a single point of failure and `onlyOwner` can use critical a few functions.

`owner` role in the project:

Owner is not behind a multisig and changes are not behind a timelock.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

`onlyOwner` functions;

    36 results - 12 files

    src/utils/MultiRewardEscrow.sol:
    207:   function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {

    src/utils/MultiRewardStaking.sol:
    251:   ) external onlyOwner {
    296:   function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {

    src/vault/AdminProxy.sol:
    21:     onlyOwner

    src/vault/CloneFactory.sol:
    39:   function deploy(Template calldata template, bytes calldata data) external onlyOwner returns (address clone) {

    src/vault/CloneRegistry.sol:
    45:   ) external onlyOwner {

    src/vault/DeploymentController.sol:
    55:   function addTemplateCategory(bytes32 templateCategory) external onlyOwner {
    80:   function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {
    103:   ) external onlyOwner returns (address clone) {
    121:   function nominateNewDependencyOwner(address _owner) external onlyOwner {

    src/vault/PermissionRegistry.sol:
    38:   function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

    src/vault/TemplateRegistry.sol:
    52:   function addTemplateCategory(bytes32 templateCategory) external onlyOwner {
    71:   ) external onlyOwner {
    102:   function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

    src/vault/Vault.sol:
    525:     function proposeFees(VaultFees calldata newFees) external onlyOwner {
    553:     function setFeeRecipient(address _feeRecipient) external onlyOwner {
    578:     function proposeAdapter(IERC4626 newAdapter) external onlyOwner {
    629:     function setQuitPeriod(uint256 _quitPeriod) external onlyOwner {
    643:     function pause() external onlyOwner {
    648:     function unpause() external onlyOwner {

    src/vault/VaultController.sol:
    408:   function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {
    543:   function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {
    561:   function addTemplateCategories(bytes32[] calldata templateCategories) external onlyOwner {
    581:     onlyOwner
    723:   function nominateNewAdminProxyOwner(address newOwner) external onlyOwner {
    751:   function setPerformanceFee(uint256 newFee) external onlyOwner {
    764:   function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {
    791:   function setHarvestCooldown(uint256 newCooldown) external onlyOwner {
    804:   function setAdapterHarvestCooldowns(address[] calldata adapters) external onlyOwner {
    832:   function setDeploymentController(IDeploymentController _deploymentController) external onlyOwner {
    864:   function setActiveTemplateId(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

    src/vault/VaultRegistry.sol:
    44:   function registerVault(VaultMetadata calldata _metadata) external onlyOwner {

    src/vault/adapter/abstracts/AdapterBase.sol:
    500:     function setHarvestCooldown(uint256 newCooldown) external onlyOwner {
    549:     function setPerformanceFee(uint256 newFee) public onlyOwner {
    574:     function pause() external onlyOwner {
    582:     function unpause() external onlyOwner {

This increases the risk of `A single point of failure`.

### **Recommended Mitigation Steps**

Add a time lock to critical functions. Admin-only functions that change critical parameters should emit events and have timelocks.

Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services.

Allow only multi-signature wallets to call the function to reduce the likelihood of an attack.

https://twitter.com/danielvf/status/1572963475101556738?s=20&t=V1kvzfJlsx-D2hfnG0OmuQ

Also detail them in documentation and NatSpec comments.

## Summary

| Number | Issues Details |
|---|---|
| [N-01] | Use of `bytes.concat()` instead of  `abi.encodePacked()` |
| [N-02] | Lock `pragma` to specific compiler version |
| [N-03] | Open TODOs |


## [N-01] Use of `bytes.concat()` instead of  `abi.encodePacked()`

    7 results - 4 files

    src/utils/MultiRewardEscrow.sol:
    104:     bytes32 id = keccak256(abi.encodePacked(token, account, amount, nonce));

    src/utils/MultiRewardStaking.sol:
    49:     _name = string(abi.encodePacked("Staked ", IERC20Metadata(address(_stakingToken)).name()));
    50:     _symbol = string(abi.encodePacked("pst-", IERC20Metadata(address(_stakingToken)).symbol()));
    51      _decimals = IERC20Metadata(address(_stakingToken)).decimals();

    460          keccak256(
    461:           abi.encodePacked(
    462              "\x19\x01",

    src/vault/Vault.sol:
    93          contractName = keccak256(
    94:             abi.encodePacked("Popcorn", name(), block.timestamp, "Vault")
    95          );

    679                  keccak256(
    680:                     abi.encodePacked(
    681                          "\x19\x01",

    src/vault/adapter/abstracts/AdapterBase.sol:
    647                  keccak256(
    648:                     abi.encodePacked(
    649                          "\x19\x01",

Rather than using `abi.encodePacked` for appending bytes, since version 0.8.4, `bytes.concat()` is enabled.

Since version 0.8.4 for appending bytes, `bytes.concat()` can be used instead of `abi.encodePacked()`.

## [N-02] Lock `pragma` to specific compiler version

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally. All of the contracts in scope use floating pragma statements.

**Recommendation:**
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.

[solidity-specific/locking-pragmas](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)

## [N-03] Open TODOs

    function _protocolDeposit(uint256 assets, uint256 shares) internal virtual {
        // OPTIONAL - convertIntoUnderlyingShares(assets,shares)
    }

    /// @notice Withdraw from the underlying protocol.
    function _protocolWithdraw(uint256 assets, uint256 shares)
        internal
        virtual
    {
        // OPTIONAL - convertIntoUnderlyingShares(assets,shares)
    }

Inside `AdapterBase.sol` there are empty block functions with comments that look like open TODOs. Consider changing them or removing them.