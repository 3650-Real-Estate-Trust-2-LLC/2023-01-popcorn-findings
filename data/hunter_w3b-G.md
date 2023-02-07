
##  ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS


The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

There are **21** instances of this issue:

#### File:/src/vaults/PermissionRegistry.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L42

**42**:     for (uint256 i = 0; i < len; i++) {


#### File: src/vault/VaultController.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L319


**319**:     for (uint8 i = 0; i < len; i++) {

**337**:     for (uint8 i = 0; i < len; i++) {

**357**:     for (uint8 i = 0; i < len; i++) {

**374**:     for (uint8 i = 0; i < len; i++) {

**437**:     for (uint256 i = 0; i < len; i++) {

**494**:     for (uint256 i = 0; i < len; i++) {

**523**:     for (uint256 i = 0; i < len; i++) {

**564**:     for (uint256 i = 0; i < len; i++) {

**587**:     for (uint256 i = 0; i < len; i++) {

**607**:     for (uint256 i = 0; i < len; i++) {

**620**:     for (uint256 i = 0; i < len; i++) {

**633**:     for (uint256 i = 0; i < len; i++) {

**646**:     for (uint256 i = 0; i < len; i++) {

**766**:     for (uint256 i = 0; i < len; i++) {

**806**:     for (uint256 i = 0; i < len; i++) {


#### File: src/utils/MultiRewardStaking.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L171


**171**:     for (uint8 i; i < _rewardTokens.length; i++) {

**373**:     for (uint8 i; i < _rewardTokens.length; i++) {


#### File: src/utils/MultiRewardEscrow.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L53

**53**:     for (uint256 i = 0; i < escrowIds.length; i++) {

**155**:     for (uint256 i = 0; i < escrowIds.length; i++) {

**210**:     for (uint256 i = 0; i < tokens.length; i++) {



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

There are **10** instances of this issue:

#### File:/src/vault/AdminProxy.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/AdminProxy.sol#L21

**19**:     function execute(address target, bytes calldata callData)
**20**:     external
**21**:     onlyOwner
**22**:     returns (bool success, bytes memory returndata)
**23**:   {



#### File:/src/vault/CloneRegistry.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L41

**41**:  function addClone(
**42**:    bytes32 templateCategory,
**43**:    bytes32 templateId,
**44**:    address clone
**45**: ) external onlyOwner {


#### File:/src/vault/DeploymentController.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L55


**99**:  function deploy(
**100**:    bytes32 templateCategory,
**101**:    bytes32 templateId,
**102**:    bytes calldata data
**103**:  ) external onlyOwner returns (address clone) {


#### File:/src/vault/TemplateRegistry.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L52


**67**:   function addTemplate(
**68**:    bytes32 templateCategory,
**69**:    bytes32 templateId,
**70**:    Template memory template
**71**:   ) external onlyOwner {


#### File:/src/utils/MultiRewardStaking.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L251

**251**:   ) external onlyOwner {


#### File:/src/vault/VaultController.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L408


**579**:   function toggleTemplateEndorsements(bytes32[] calldata templateCategories, bytes32[] calldata templateIds)
**580**:    external
**581**:    onlyOwner

#### File:/src/vault/adapter/abstracts/AdapterBase.sol

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L574

**500**:     function setHarvestCooldown(uint256 newCooldown) external onlyOwner {

**549**:     function setPerformanceFee(uint256 newFee) public onlyOwner {

**574**:     function pause() external onlyOwner {

**582**:    function unpause() external onlyOwner {



