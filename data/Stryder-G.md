
Popcorn 

Gas Optimizations : 

MultiRewardEscrow.sol : 

Line 53: instead of doing escrowIds.length you can store the length of escrowIds.length in a uint to save some gas fee. In this case, the length is retaken from the array that takes up more gas than necessary  , something like this  

Uint n = escrowIds.length
Then use the n in the for loop

Line 53: ++i is a better way of incrementing i as it saves some more gas 

Line 154 : 

You can make the escrowIds parameter as calldata to save some more gas as this is only used to view so there is no benefit of using memory in this one. Something like this  : 
function claimRewards(bytes32[] calldata escrowIds)


Line 155: instead of doing escrowIds.length you can store the length of escrowIds.length in a uint to save some gas fee. In this case, the length is retaken from the array that takes up more gas than necessary  , something like this  

Uint n = escrowIds.length
Then use the n in the for loop

Line 155: ++i is a better way of incrementing i as it saves some more gas 

Line 211 :

Instead of using >= operator you can save some more gas by just using >  and putting one more value on the right hand side .
So you can do something like this 

  if (tokenFees[i] > 100000000000000001) revert DontGetGreedy(tokenFees[i]); 


MultiRewardStaking

Line  170 : 


You can make the _rewardTokens parameter as calldata to save some more gas as this is only used to view so there is no benefit of using memory in this one. Something like this  : 
function claimRewards(address user, IERC20[] calldata _rewardTokens)

Line 171 : instead of doing _rewardTokens.length you can store the length of _rewardTokens.length in a uint to save some gas fee. In this case, the length is retaken from the array that takes up more gas than necessary  , something like this  

Uint n = _rewardTokens.length
Then use the n in the for loop

VaultController 

Line 36 -40 : 
Immutable is usually used to assign values in the constructor, but since these values are being declared and assigned in the contract's main body, so you can directly initialize them as constant. Sort of like this 

  bytes32 public constant VAULT = "Vault";
  bytes32 public constant ADAPTER = "Adapter";
  bytes32 public constant STRATEGY = "Strategy";
  bytes32 public constant STAKING = "Staking";
  bytes4 internal constant DEPLOY_SIG = bytes4(keccak256("deploy(bytes32,bytes32,bytes)"));


Line 242 : 

The parameter are just being viewed only so you can write them as calldata , like this 

  function _encodeAdapterData(DeploymentArgs calldata adapterData, bytes  calldata baseAdapterData)

Line 256: 

The parameter are just being viewed only so you can write them as calldata , like this 

function _deployStrategy(DeploymentArgs calldata strategyData, IDeploymentController _deploymentController)
