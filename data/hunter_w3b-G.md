INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

There are 14 instances of this issue: 


File: src/utils/MultiRewardStaking.sol

191:  function _lockToken(
192:    address user,
193:    IERC20 rewardToken,
194:    uint256 rewardAmount,
195:    EscrowInfo memory escrowInfo
196:  )  internal {



File: src/vault/adapter/yearn/YearnAdapter.sol

89:  function _shareValue(uint256 yShares) internal


101:  function _freeFunds() internal view returns (uint256) {


109:    function _yTotalAssets() internal view returns (uint256) {


114:   function _calculateLockedProfit() internal view returns (uint256) {



File: src/vault/VaultController.sol

120: function _deployVault(VaultInitParams memory vaultData, IDeploymentController _deploymentController)
121:    internal

141:  function _registerCreatedVault(
142:    address vault,
143:    address staking,
144:    VaultMetadata memory metadata
145:  ) internal  {

154:  function _handleVaultStakingRewards(address vault, bytes memory rewardsData) internal {

225:  function __deployAdapter(
226:    DeploymentArgs memory adapterData,
227:    bytes memory baseAdapterData,
228:    IDeploymentController _deploymentController
229:  ) internal returns (address adapter) {

242:   function _encodeAdapterData(DeploymentArgs memory adapterData, bytes memory baseAdapterData)
243:    internal

256:   function _deployStrategy(DeploymentArgs memory strategyData, IDeploymentController _deploymentController)
257:    internal

390:   function _registerVault(address vault, VaultMetadata memory metadata) internal {

692:   function _verifyAdapterConfiguration(address adapter, bytes32 adapterId) internal view {



File:/src/vault/adapter/abstracts/AdapterBase.sol

479:    function _verifyAndSetupStrategy(bytes4[8] memory requiredSigs) internal {
