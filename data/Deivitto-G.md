
# Gas

| | Issue index |
| ----------- | ----------- |
| 1 | [Variable can be declared immutable](#variable-can-be-declared-immutable) |
| 2 | [Variable can be declared constant](#variable-can-be-declared-constant) |
| 3 | [Multiple `address`/`ID` `mappings` can be combined into a single `mapping` of an `address`/`ID` to a `struct`, where appropriate](#multiple-`address`/`id`-`mappings`-can-be-combined-into-a-single-`mapping`-of-an-`address`/`id`-to-a-`struct`,-where-appropriate) |
| 4 | [state variables can be packed into fewer storage slots](#state-variables-can-be-packed-into-fewer-storage-slots) |
| 5 | [`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables](#`<x>-+=-<y>`-costs-more-gas-than-`<x>-=-<x>-+-<y>`-for-state-variables) |
| 6 | [`++i / i++` should be `unchecked{++i} / unchecked{i++}` when it is not possible for them to overflow, as is the case when used in for-loop and while-loops](#`++i-/-i++`-should-be-`unchecked{++i}-/-unchecked{i++}`-when-it-is-not-possible-for-them-to-overflow,-as-is-the-case-when-used-in-for-loop-and-while-loops) |
| 7 | [`internal` functions only called once can be inlined to save gas](#`internal`-functions-only-called-once-can-be-inlined-to-save-gas) |
| 8 | [Cache variables](#cache-variables) |
| 9 | [Should use arguments instead of state variable](#should-use-arguments-instead-of-state-variable) |
| 10 | [Superfluous event fields](#superfluous-event-fields) |
| 11 | [`abi.encode()` is less gas efficient than `abi.encodePacked()`](#`abi.encode()`-is-less-gas-efficient-than-`abi.encodepacked()`) |


## Variable can be declared immutable 

It's only assigned at constructor

- [MultiRewardEscrow.sol#L191](https://github.com/code-423n4/2023-01-popcorn/blob/7a513a9734b9e49af33041e2032ffc131f3b73b0/src/utils/MultiRewardEscrow.sol#L191)

## Variable can be declared constant 

Variables that do not change and not assigned in the constructor should be constant 

`assetsCheckpoint`:

- [Vault.sol#L467](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L467)

`FEE_RECIPIENT`:

- [AdapterBase.sol#L517](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L517)

Some immutables that can be constant:

- [VaultController.sol#L36-L39](https://github.com/code-423n4/2023-01-popcorn/blob/7a513a9734b9e49af33041e2032ffc131f3b73b0/src/vault/VaultController.sol#L36-L39)


## Multiple `address`/`ID` `mappings` can be combined into a single `mapping` of an `address`/`ID` to a `struct`, where appropriate

If both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s `keccak256` hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. 

- `Template categories` / `ids`:

- [TemplateRegistry.sol#L31](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L31)
  mapping(bytes32 => mapping(bytes32 => Template)) public templates;

- [TemplateRegistry.sol#L32](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L32)
  mapping(bytes32 => bytes32[]) public templateIds;

- [TemplateRegistry.sol#L33](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L33)
  mapping(bytes32 => bool) public templateExists;

- [TemplateRegistry.sol#L35](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L35)
  mapping(bytes32 => bool) public templateCategoryExists;


- `Users`:

- [MultiRewardEscrow.sol#L67](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L67)
  mapping(address => bytes32[]) public userEscrowIds;

- [MultiRewardEscrow.sol#L69](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L69)
  mapping(address => mapping(IERC20 => bytes32[])) public userEscrowIdsByToken;


- `rewardToken`:

- [MultiRewardStaking.sol#L211](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L211)
  mapping(IERC20 => RewardInfo) public rewardInfos;

- [MultiRewardStaking.sol#L213](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L213)
  mapping(IERC20 => EscrowInfo) public escrowInfos;


- `users`:

- [MultiRewardStaking.sol#L216](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L216)
  mapping(address => mapping(IERC20 => uint256)) public userIndex;

- [MultiRewardStaking.sol#L218](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L218)
  mapping(address => mapping(IERC20 => uint256)) public accruedRewards;


## state variables can be packed into fewer storage slots

Consider joining in a single slot following variables

- [AdapterBase.sol#L38](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L38)
uint8 internal _decimals;

- [AdapterBase.sol#L517](https://github.com/code-423n4/2023-01-popcorn/blob/dcdd3ceda3d5bd87105e691ebc054fb8b04ae583/src/vault/adapter/abstracts/AdapterBase.sol#L517)
address FEE_RECIPIENT = address(0x4444);

## `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

Using the addition operator instead of plus-equals saves gas. Same goes for other operands like `-=`.

- [MultiRewardStaking.sol#L408](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L408)
    rewardInfos[_rewardToken].index += deltaIndex;

- [MultiRewardStaking.sol#L431](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L431)
    accruedRewards[_user][_rewardToken] += supplierDelta;

- [AdapterBase.sol#L158](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L158)
        underlyingBalance += _underlyingBalance() - underlyingBalance_;

- [MultiRewardEscrow.sol#L162](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L162)
      escrows[escrowId].balance -= claimable;

- [AdapterBase.sol#L225](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L225)
            underlyingBalance -= underlyingBalance_ - _underlyingBalance();

## `++i / i++` should be `unchecked{++i} / unchecked{i++}` when it is not possible for them to overflow, as is the case when used in for-loop and while-loops

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

- [PermissionRegistry.sol#L42](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L42)
    for (uint256 i = 0; i < len; i++) {

- [MultiRewardEscrow.sol#L53](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L53)
    for (uint256 i = 0; i < escrowIds.length; i++) {

- [MultiRewardEscrow.sol#L155](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L155)
    for (uint256 i = 0; i < escrowIds.length; i++) {

- [MultiRewardEscrow.sol#L210](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L210)
    for (uint256 i = 0; i < tokens.length; i++) {

- [MultiRewardStaking.sol#L171](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L171)
    for (uint8 i; i < _rewardTokens.length; i++) {

- [MultiRewardStaking.sol#L373](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L373)
    for (uint8 i; i < _rewardTokens.length; i++) {

- [VaultController.sol#L319](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L319)
    for (uint8 i = 0; i < len; i++) {

- [VaultController.sol#L337](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L337)
    for (uint8 i = 0; i < len; i++) {

- [VaultController.sol#L357](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L357)
    for (uint8 i = 0; i < len; i++) {

- [VaultController.sol#L374](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L374)
    for (uint8 i = 0; i < len; i++) {

- [VaultController.sol#L437](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L437)
    for (uint256 i = 0; i < len; i++) {

- [VaultController.sol#L494](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L494)
    for (uint256 i = 0; i < len; i++) {

- [VaultController.sol#L523](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L523)
    for (uint256 i = 0; i < len; i++) {

- [VaultController.sol#L564](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L564)
    for (uint256 i = 0; i < len; i++) {

- [VaultController.sol#L587](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L587)
    for (uint256 i = 0; i < len; i++) {

- [VaultController.sol#L607](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L607)
    for (uint256 i = 0; i < len; i++) {

- [VaultController.sol#L620](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L620)
    for (uint256 i = 0; i < len; i++) {

- [VaultController.sol#L633](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L633)
    for (uint256 i = 0; i < len; i++) {

- [VaultController.sol#L646](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L646)
    for (uint256 i = 0; i < len; i++) {

- [VaultController.sol#L766](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L766)
    for (uint256 i = 0; i < len; i++) {

- [VaultController.sol#L806](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L806)
    for (uint256 i = 0; i < len; i++) {

## `internal` functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

- [YearnAdapter.sol#L89](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L89)
    function _shareValue(uint256 yShares) internal view returns (uint256) {

- [YearnAdapter.sol#L101](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L101)
    function _freeFunds() internal view returns (uint256) {

- [YearnAdapter.sol#L109](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L109)
    function _yTotalAssets() internal view returns (uint256) {

- [YearnAdapter.sol#L114](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L114)
    function _calculateLockedProfit() internal view returns (uint256) {

- [VaultController.sol#L154](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L154)
  function _handleVaultStakingRewards(address vault, bytes memory rewardsData) internal {

- [VaultController.sol#L390](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L390)
  function _registerVault(address vault, VaultMetadata memory metadata) internal {

## Cache variables 

Avoid innecesary SLOADs

emit can be called at the end with a local variable `_nominatedOwner` that caches 3 times accesed `nominatedOwner`

- [Owned.sol#L24](https://github.com/code-423n4/2023-01-popcorn/blob/7a513a9734b9e49af33041e2032ffc131f3b73b0/src/utils/Owned.sol#L24)

- [OwnedUpgradeable.sol#L24-L28](https://github.com/code-423n4/2023-01-popcorn/blob/7a513a9734b9e49af33041e2032ffc131f3b73b0/src/utils/OwnedUpgradeable.sol#L24-L28)

Same issue:

- [Vault.sol#L544-L545](https://github.com/code-423n4/2023-01-popcorn/blob/7a513a9734b9e49af33041e2032ffc131f3b73b0/src/vault/Vault.sol#L544-L545)

- [Vault.sol#L606](https://github.com/code-423n4/2023-01-popcorn/blob/7a513a9734b9e49af33041e2032ffc131f3b73b0/src/vault/Vault.sol#L606) 

- [VaultController.sol#L837-L840](https://github.com/code-423n4/2023-01-popcorn/blob/7a513a9734b9e49af33041e2032ffc131f3b73b0/src/vault/VaultController.sol#L837-L840)

## Should use arguments instead of state variable

This will save near 97 gas

- [Vault.sol#L97](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L97)
        emit VaultInitialized(contractName, address(asset));

- [Vault.sol#L635](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L635)
        emit QuitPeriodSet(quitPeriod);

## Superfluous event fields

`block.timestamp` and `block.number` are added to event information by default so adding them manually wastes extra gas.

- [Vault.sol#L536](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L536)
        emit NewFeesProposed(newFees, block.timestamp);

- [Vault.sol#L585](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L585)
        emit NewAdapterProposed(newAdapter, block.timestamp);

## `abi.encode()` is less gas efficient than `abi.encodePacked()`

Changing the abi.encode function to `abi.encodePacked` can save gas since the `abi.encode` function pads extra null `bytes` at the end of the call data, which is unnecessary. Also, in general, `abi.encodePacked` is more gas-efficient.

- [MultiRewardStaking.sol#L465](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L465)
              abi.encode(

- [MultiRewardStaking.sol#L494](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L494)
        abi.encode(

- [Vault.sol#L684](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L684)
                            abi.encode(

- [Vault.sol#L719](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L719)
                abi.encode(

- [VaultController.sol#L219](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L219)
        abi.encode(asset, address(adminProxy), IStrategy(strategy), harvestCooldown, requiredSigs, strategyData.data),

- [AdapterBase.sol#L652](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L652)
                            abi.encode(

- [AdapterBase.sol#L687](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L687)
                abi.encode(
