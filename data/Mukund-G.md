| Index | Issue                                                                                    |
|-------|------------------------------------------------------------------------------------------|
| 1     | <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables                       |
| 2     | abi.encode() is less efficient than abi.encodepacked()                                   |
| 3     | Public Functions To External                                                             |
| 4     | Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead                     |
| 5     | Setting the constructor to payable                                                       |
| 6     | Functions guaranteed to revert when called by normal users can be marked payable         |
| 7     | Use uncheck block for operation that can not overflow/underflow                          |
| 8     | Unused function should be removed                                                        |
| 9     | Use if else block rather than function                                                   |
| 10    | Using calldata instead of memory for read-only arguments in external functions saves gas |
## 1.<x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables
[Vault.sol#L228](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L228)
```
   shares += feeShares;
```

## 2. abi.encode() is less efficient than abi.encodepacked()
See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison
```
abi.encode(
```
[MultiRewardStaking.sol#L465](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L465)
[MultiRewardStaking.sol#L494](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L494)
[Vault.sol#L684](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L684)
[Vault.sol#L719](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L719)
[VaultController.sol#L219](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L219)
[AdapterBase.sol#L652](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L652)
[AdapterBase.sol#L687](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L687)

## 3. Public Functions To External
The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.
```
    function maxDeposit(address caller) public view returns (uint256) {
        return adapter.maxDeposit(caller);
    }
```
[Vault.sol#L399](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L399)
[YearnAdapter.sol#L144](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L144)

## 4. Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.15/internals/layout_in_storage.html Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed
```
    uint32 duration,
    uint32 offset
```
[MultiRewardEscrow.sol#L92](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L92)
[MultiRewardEscrow.sol#L114](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L114)
[MultiRewardStaking.sol#L33](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L33)
[MultiRewardStaking.sol#L171](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L171)
[MultiRewardStaking.sol#L220](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L220)
[MultiRewardStaking.sol#L245](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L245)
[MultiRewardStaking.sol#L248-L250](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L248-L250)
[MultiRewardStaking.sol#L274-L275](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L274-L275)
[MultiRewardStaking.sol#L307-L308](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L307-L308)
[MultiRewardStaking.sol#L352-L353](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L352-L353)
[Vault.sol#L38](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L38)
[Vault.sol#L669](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L669)
[VaultController.sol#L314](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L314)
[VaultController.sol#L319](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L319)
[VaultController.sol#L336-L337](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L336-L337)
[VaultController.sol#L353](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L353)
[VaultController.sol#L357](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L357)
[VaultController.sol#L373-L374](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L373-L374)

## 5. Setting the constructor to payable
Saves ~13 gas per instance
```
  constructor(
    address _owner,
    ICloneFactory _cloneFactory,
    ICloneRegistry _cloneRegistry,
    ITemplateRegistry _templateRegistry
  ) Owned(_owner) {
    cloneFactory = _cloneFactory;
    cloneRegistry = _cloneRegistry;
    templateRegistry = _templateRegistry;
  }
```
[VaultController.sol#L53-L70](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L53-L70)
[CloneFactory.sol#L23](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneFactory.sol#L23)
[CloneRegistry.sol#L22](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L22)
[VaultRegistry.sol#L21](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultRegistry.sol#L21)
[DeploymentController.sol#L35-L44](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L35-L44)
[TemplateRegistry.sol#L24](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L24)
[MultiRewardEscrow.sol#L30](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L30)
[VaultController.sol#L53-L70](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L53-L70)

## 6. Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.
```
  function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {
```
[MultiRewardEscrow.sol#L207](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L207)
[MultiRewardStaking.sol#L243-L251](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L243-L251)
[MultiRewardStaking.sol#L296](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L296)
[CloneFactory.sol#L39](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneFactory.sol#L39)
[CloneRegistry.sol#L41-L45](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneRegistry.sol#L41-L45)
[DeploymentController.sol#L55](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L55)
[DeploymentController.sol#L80](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L80)
[DeploymentController.sol#L99-L103](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L99-L103)
[DeploymentController.sol#L121](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L121)
[PermissionRegistry.sol#L38](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L38)
[TemplateRegistry.sol#L52](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L52)
[TemplateRegistry.sol#L67-L71](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L67-L71)
[TemplateRegistry.sol#L102](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L102)
[Vault.sol#L525](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L525)
[Vault.sol#L553](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L553)
[Vault.sol#L578](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L578)
[Vault.sol#L629](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L629)
[VaultController.sol#L408](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L408)
[VaultController.sol#L543](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L543)
[VaultController.sol#L723](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L723)
[VaultController.sol#L751](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L751)
[VaultController.sol#L764](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L764)
[VaultController.sol#L791](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L791)
[VaultController.sol#L804](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L804)
[VaultController.sol#L864](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L864)

## 7. Use uncheck block for operation that can not overflow/underflow
using uncheck block for i++ in every loop can save gas
```
for (uint256 i = 0; i < escrowIds.length; i++) {
```
[MultiRewardEscrow.sol#L53](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L53)
[MultiRewardEscrow.sol#L155](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L155)
[MultiRewardEscrow.sol#L210](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L210)
[MultiRewardStaking.sol#L171](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L171)
[MultiRewardStaking.sol#L373](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L373)
[PermissionRegistry.sol#L42](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L42)
[VaultController.sol#L319](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L319)
[VaultController.sol#L337](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L337)
[VaultController.sol#L357](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L357)
[VaultController.sol#L374](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L374)
[VaultController.sol#L437](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L437)
[VaultController.sol#L494](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L494)
[VaultController.sol#L523](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L523)
[VaultController.sol#L564](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L564)
[VaultController.sol#L587](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L587)
[VaultController.sol#L607](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L607)
[VaultController.sol#L620](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L620)
[VaultController.sol#L633](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L633)
[VaultController.sol#L646](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L646)
[VaultController.sol#L766](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L766)
[VaultController.sol#L806](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L806)

## 8. Unused function should be removed
```
    function takeManagementAndPerformanceFees()
        external
        nonReentrant
        takeFees
    {}
```
[Vault.sol#L473](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L473)

## 9. Use if else block rather than function
use if block to validate array lengths rather than defining a separate function to validate array length , if block is cheaper than calling a function which has if block.
```
   _verifyEqualArrayLength( length1,  length2);

  function _verifyEqualArrayLength(uint256 length1, uint256 length2) internal pure {
    if (length1 != length2) revert ArrayLengthMissmatch();
  }
```
[VaultController.sol#L316](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L316)
[VaultController.sol#L355](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L355)
[VaultController.sol#L434](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L434)
[VaultController.sol#L490](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L490)
[VaultController.sol#L491](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L491)
[VaultController.sol#L519-L520](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L519-L520)
[VaultController.sol#L544](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L544)
[VaultController.sol#L584](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L584)
[VaultController.sol#L700](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L700)

## 10. Using calldata instead of memory for read-only arguments in external functions saves gas
When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it's still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it's still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one
```
  function getClonesByCategoryAndId(bytes32 templateCategory, bytes32 templateId)
    external
    view
    returns (address[] memory)
  {
    return clones[templateCategory][templateId];
  }
```
[CloneRegistry.sol#L57-L68](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneRegistry.sol#L57-L68)
[TemplateRegistry.sol#L115-L121](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L115-L121)
[VaultRegistry.sol#L63-L65](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L63-L65)
[VaultRegistry.sol#L71-L73](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L71-L73)
[BeefyAdapter.sol#L136](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L136)








