## [L-01] Consider Add Boundry for `rewardPerSecond" to avoid overflow

When `addToken` and `changeRewardSpeed`, there is no boundry for `rewardPerSecond`. Maybe some basic boundry should be added.

```solidity
File: src/utils/MultiRewardStaking.sol
function addRewardToken(
    IERC20 rewardToken,
    uint160 rewardsPerSecond,
    uint256 amount,
    bool useEscrow,
    uint192 escrowPercentage,
    uint32 escrowDuration,
    uint32 offset
  ) external onlyOwner

function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner
```
## [L-02] Invalid Permission Control for `addTemplate`

As the comment of theser two function says: `* @notice Adds a new category for templates. Caller must be owner. ` But in below code snippet, there is no control over caller:
```solidity
File: src/vault/DeploymentController.sol
  function addTemplate(
    bytes32 templateCategory,
    bytes32 templateId,
    Template calldata template
  ) external {
    templateRegistry.addTemplate(templateCategory, templateId, template);
  }
```

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L66

Since the comment here says it should only be called by owner, and in the `TemplateRegistry.sol`, this function has the modifier `onlyOwner` where the owner is just the `DeploymentController` above. Missing modifier here will make the permisson control in `TemplateRegistry` useless.

## [NC-01] Wrong Comment for BPS
When `1e18=100%`, `1 BPS` should be `1e14` as other comments of this file says.
```solidity
File: src/vault/ValutController.sol
      /**
       * @notice Sets new fees per vault. Caller must be creator of the vaults.
       * @param vaults Addresses of the vaults to change
       * @param fees New fee structures for these vaults
       * @dev Value is in 1e18, e.g. 100% = 1e18 - 1 BPS = 1e12
       */
```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L350

## [NC-02] Confusion caused by repeated use of name `owner`

Parameter name `owner` is used here:
```solidity
File: src/utils/MultiRewardStaking.sol
121   function _withdraw(
        address caller,
        address receiver,
        address owner,
        uint256 assets,
        uint256 shares
      ) internal override accrueRewards(caller, receiver) {
        if (caller != owner) _approve(owner, msg.sender, allowance(owner, msg.sender) - shares);

        _burn(owner, shares);
        IERC20(asset()).safeTransfer(receiver, assets);

        emit Withdraw(caller, receiver, owner, assets, shares);
      }
```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L121

However, since this contract is inherited from `OwnedUpgradeable`, where there is also a state variable named `owner`:
```solidity
contract OwnedUpgradeable is Initializable {
  address public owner;
  address public nominatedOwner;
    ...
```

Consider change the parameter name of function `_withdraw`.

## [NC-03] Code length exceeds 80 letters
Just list some of them below for example:
```solidity
File: src/utils/MultiRewardStaking.sol
    _name = string(abi.encodePacked("Staked ", IERC20Metadata(address(_stakingToken)).name()));

    _symbol = string(abi.encodePacked("pst-", IERC20Metadata(address(_stakingToken)).symbol()));

    function name() public view override(ERC20Upgradeable, IERC20Metadata) returns (string memory) {
    
    function claimRewards(address user, IERC20[] memory _rewardTokens) external accrueRewards(msg.sender, user) {
```
