## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Missing checks for `address(0)` when assigning values to address state variables | 1 |
| [NC-2](#NC-2) | Return values of `approve()` not checked | 12 |
| [NC-3](#NC-3) | Function can be restricted to view | 1 |
| [NC-4](#NC-4) | Less Event Paramenter | 1 |
### <a name="NC-1"></a>[NC-1] Missing checks for `address(0)` when assigning values to address state variables

*Instances (1)*:
```solidity
File: src/utils/MultiRewardEscrow.sol

31:     feeRecipient = _feeRecipient;

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol)

### <a name="NC-2"></a>[NC-2] Return values of `approve()` not checked
Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

*Instances (12)*:
```solidity
File: src/utils/MultiRewardStaking.sol

483:       _approve(recoveredAddress, spender, value);

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol)

```solidity
File: src/vault/Vault.sol

80:         asset.approve(address(adapter_), type(uint256).max);

231:             _approve(owner, msg.sender, allowance(owner, msg.sender) - shares);

261:             _approve(owner, msg.sender, allowance(owner, msg.sender) - shares);

604:         asset.approve(address(adapter), 0);

610:         asset.approve(address(adapter), type(uint256).max);

705:             _approve(recoveredAddress, spender, value);

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol)

```solidity
File: src/vault/VaultController.sol

171:       asset.approve(address(target), initialDeposit);

456:       IERC20(rewardsToken).approve(staking, type(uint256).max);

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol)

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

80:         IERC20(asset()).approve(_beefyVault, type(uint256).max);

83:             IERC20(_beefyVault).approve(_beefyBooster, type(uint256).max);

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol)

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

54:         IERC20(_asset).approve(address(yVault), type(uint256).max);

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol)


### <a name="NC-3"></a>[NC-3] Function can be restricted to view

*Instances (1)*:
```solidity
File: src/vault/VaultController.sol

667:   function _verifyCreatorOrOwner(address vault) internal returns (VaultMetadata memory metadata) {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/vaultController.sol)



### <a name="NC-4"></a>[NC-4] Less Event Paramenter
Emmiting both oldendorsement and !oldendorsement is not necessary as either of the values can be deduced from the other.

And also it cost more gas to emit 4 parameters in an event while you need less gas when you emit just three parameters in an event.

*Instances (1)*:
```solidity
File: src/vault/TemplateRegistry.sol

108: emit TemplateEndorsementToggled(templateCategory, templateId, oldEndorsement, !oldEndorsement);

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol)



## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Empty Function Body | 10 |
| [L-2](#L-2) | Initializers could be front-run | 14 |
| [L-3](#L-3) | Low level call risk | 1 |
| [L-4](#L-4) | Floating Pragma | 24 |

### <a name="L-1"></a>[L-1] Empty Function Body - Consider commenting why

An empty function block in Solidity can introduce a number of vulnerabilities in smart contracts if not used correctly. Some of the most common vulnerabilities associated with empty function blocks are:

1. Fallback functions: An empty function block can be interpreted as a fallback function, which can be executed if no other function matches the incoming transaction data. An attacker could exploit this by sending malicious transactions to the contract that trigger the fallback function.

2. Unintended execution: An empty function block can be inadvertently executed due to a bug in the calling code, leading to unintended behavior or security vulnerabilities.

3. Gas costs: An empty function block will still consume gas for its execution, even though it does not perform any action. This can lead to excessive gas costs and limit the overall functionality of the contract.

To mitigate these vulnerabilities, it is recommended to follow best practices such as:

1. Avoid using empty function blocks whenever possible.

2. If a fallback function is required, implement it with a revert statement to prevent unintended execution.

*Instances (10)*:
```solidity
File: src/utils/EIP165.sol

7:   function supportsInterface(bytes4 interfaceId) public view virtual returns (bool) {}

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/EIP165.sol)

```solidity
File: src/vault/AdminProxy.sol

16:   constructor(address _owner) Owned(_owner) {}

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/AdminProxy.sol)

```solidity
File: src/vault/CloneFactory.sol

23:   constructor(address _owner) Owned(_owner) {}

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneFactory.sol)

```solidity
File: src/vault/CloneRegistry.sol

22:   constructor(address _owner) Owned(_owner) {}

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol)

```solidity
File: src/vault/PermissionRegistry.sol

20:   constructor(address _owner) Owned(_owner) {}

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol)

```solidity
File: src/vault/TemplateRegistry.sol

24:   constructor(address _owner) Owned(_owner) {}

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol)

```solidity
File: src/vault/Vault.sol

477:     {}

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol)

```solidity
File: src/vault/VaultRegistry.sol

21:   constructor(address _owner) Owned(_owner) {}

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol)

```solidity
File: src/vault/adapter/abstracts/WithRewards.sol

13:   function rewardTokens() external view virtual returns (address[] memory) {}

15:   function claim() public virtual onlyStrategy {}

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/WithRewards.sol)

### <a name="L-2"></a>[L-2] Initializers could be front-run
Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment

*Instances (14)*:
```solidity
File: src/utils/MultiRewardStaking.sol

41:   function initialize(

45:   ) external initializer {

46:     __ERC4626_init(IERC20Metadata(address(_stakingToken)));

47:     __Owned_init(_owner);

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol)

```solidity
File: src/vault/Vault.sol

57:     function initialize(

63:     ) external initializer {

64:         __ERC20_init(

72:         __Owned_init(owner);

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol)

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

46:     function initialize(

50:     ) external initializer {

55:         __AdapterBase_init(adapterInitData);

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol)

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

34:     function initialize(

38:     ) external initializer {

43:         __AdapterBase_init(adapterInitData);

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol)

### <a name="L-3"></a>[L-3] Low level call risk
Consider Implementing the "pull over push" pattern, where the target contract pulls data from the calling contract instead of the calling contract pushing data to the target contract.

*Instances (1)*:
```solidity
File: src/vault/AdminProxy.sol

19:   function execute(address target, bytes calldata callData) external onlyOwner returns (bool success, bytes memory returndata) {

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/AdminProxy.sol)


### <a name="L-4"></a>[L-4] Floating Pragma
Consider Implementing the "pull over push" pattern, where the target contract pulls data from the calling contract instead of the calling contract pushing data to the target contract.

*Instances (24)*:
```solidity
File: All contract in src/vault and src/utils

4:   pragma solidity ^0.8.15;

```
[Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault) and [Link to code](https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils)

