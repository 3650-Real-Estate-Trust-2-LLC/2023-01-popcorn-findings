## INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS
Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

There are **14** instances of this issue: 


#### File: src/utils/MultiRewardStaking.sol
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L191

**191**:  function _lockToken(
**192**:    address user,
**193**:    IERC20 rewardToken,
**194**:    uint256 rewardAmount,
**195**:    EscrowInfo memory escrowInfo
**196**:  )  internal {



#### File: src/vault/adapter/yearn/YearnAdapter.sol
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L89

**89**:  function _shareValue(uint256 yShares) internal


**101**:  function _freeFunds() internal view returns (uint256) {


**109**:    function _yTotalAssets() internal view returns (uint256) {


**114**:   function _calculateLockedProfit() internal view returns (uint256) {



#### File: src/vault/VaultController.sol
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L120

**120**: function _deployVault(VaultInitParams memory vaultData, IDeploymentController _deploymentController)
**121**:    internal

**141**:  function _registerCreatedVault(
**142**:    address vault,
**143**:    address staking,
**144**:    VaultMetadata memory metadata
**145**:  ) internal  {

**154**:  function _handleVaultStakingRewards(address vault, bytes memory rewardsData) internal {

**225**:  function __deployAdapter(
**226**:    DeploymentArgs memory adapterData,
**227**:    bytes memory baseAdapterData,
**228**:    IDeploymentController _deploymentController
**229**:  ) internal returns (address adapter) {

**242**:   function _encodeAdapterData(DeploymentArgs memory adapterData, bytes memory baseAdapterData)
**243**:    internal

**256**:   function _deployStrategy(DeploymentArgs memory strategyData, IDeploymentController _deploymentController)
**257**:    internal

**390**:   function _registerVault(address vault, VaultMetadata memory metadata) internal {

**692**:   function _verifyAdapterConfiguration(address adapter, bytes32 adapterId) internal view {



#### File:/src/vault/adapter/abstracts/AdapterBase.sol
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L479

**479**:    function _verifyAndSetupStrategy(bytes4[8] memory requiredSigs) internal {


## ADD UNCHECKED {} FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS  IF-STATEMENT

require( a < b );    =>   unchecked { x = b - a }
if( a < b ){return a+b;}     =>  if (a <=b) { **unchecked{ return a + b; }** }

There is **3** instance of this issue:

#### File:/src/vault/adapter/yearn/YearnAdapter.sol  
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L120

  **120**  :      return
  **121** :               lockedProfit -
  **122**:                ((lockedFundsRatio * lockedProfit) / DEGRADATION_COEFFICIENT); 


#### File:/src/utils/MultiRewardStaking.sol
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L357


**357** :    amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);

#### File:/src/vault/Vault.sol
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L455


**455** :                   (shareValue - highWaterMark) * totalSupply(),

## FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

There are **35** instances of this issue:

#### File:/src/vault/AdminProxy.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L21

**19**:     function execute(address target, bytes calldata callData)
**20**:     external
**21**:     onlyOwner
**22**:     returns (bool success, bytes memory returndata)
**23**:   {

#### File:/src/vault/CloneFactory.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneFactory.sol#L39

**39**:  function deploy(Template calldata template, bytes calldata data) external onlyOwner returns (address clone) {



#### File:/src/vault/PermissionRegistry.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L38

**38**:  function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

#### File:/src/vault/CloneRegistry.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L41

**41**:  function addClone(
**42**:    bytes32 templateCategory,
**43**:    bytes32 templateId,
**44**:    address clone
**45**: ) external onlyOwner {

#### File:/src/vault/VaultRegistry.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultRegistry.sol#L44

**44**:  function registerVault(VaultMetadata calldata _metadata) external onlyOwner {



#### File:/src/vault/DeploymentController.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L55

**55**:  function addTemplateCategory(bytes32 templateCategory) external onlyOwner {

**80**:   function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

**99**:  function deploy(
**100**:    bytes32 templateCategory,
**101**:    bytes32 templateId,
**102**:    bytes calldata data
**103**:  ) external onlyOwner returns (address clone) {

**121**:   function nominateNewDependencyOwner(address _owner) external onlyOwner {


#### File:/src/vault/TemplateRegistry.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L52

**52**:   function addTemplateCategory(bytes32 templateCategory) external onlyOwner {

**67**:   function addTemplate(
**68**:    bytes32 templateCategory,
**69**:    bytes32 templateId,
**70**:    Template memory template
**71**:   ) external onlyOwner {


**102**:   function toggleTemplateEndorsement(bytes32 templateCategory, bytes32 templateId) external onlyOwner {


#### File:/src/utils/MultiRewardEscrow.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L207

**207**:  function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {


#### File:/src/utils/MultiRewardStaking.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L251

**251**:   ) external onlyOwner {

**296**:   function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {

#### File:/src/vault/Vault.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L525

**525**:     function proposeFees(VaultFees calldata newFees) external onlyOwner {

**553**:     function setFeeRecipient(address _feeRecipient) external onlyOwner {

**578**:     function proposeAdapter(IERC4626 newAdapter) external onlyOwner {

**629**:     function setQuitPeriod(uint256 _quitPeriod) external onlyOwner {

**643**:     function pause() external onlyOwner {

**648**:     function unpause() external onlyOwner {

#### File:/src/vault/VaultController.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L408

**408**:  function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

**543**:    function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {

**561**:   function addTemplateCategories(bytes32[] calldata templateCategories) external onlyOwner {

**579**:   function toggleTemplateEndorsements(bytes32[] calldata templateCategories, bytes32[] calldata templateIds)
**580**:    external
**581**:    onlyOwner

**723**:   function nominateNewAdminProxyOwner(address newOwner) external onlyOwner {

**751**:   function setPerformanceFee(uint256 newFee) external onlyOwner {

**764**:   function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {

**804**:   function setAdapterHarvestCooldowns(address[] calldata adapters) external onlyOwner {

**832**:   function setDeploymentController(IDeploymentController _deploymentController) external onlyOwner {

**864**:   function setActiveTemplateId(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

#### File:/src/vault/adapter/abstracts/AdapterBase.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L574

**500**:     function setHarvestCooldown(uint256 newCooldown) external onlyOwner {

**549**:     function setPerformanceFee(uint256 newFee) public onlyOwner {

**574**:     function pause() external onlyOwner {

**582**:    function unpause() external onlyOwner {


