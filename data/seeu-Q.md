## [L-01] TODO comments in the code base

### Description

Before deploying the system, the following instances of TODO comments in the codebase should be noted in the project's problems backlog and eliminated.

Having clearly stated TODO notes throughout development will facilitate monitoring and resolving them. Without that knowledge, these remarks could become outdated and crucial details for the system's security might be lost by the time it is put into production.

Consider keeping track of all TODO comments in the backlog of issues and connecting each inline TODO to the related item. Before deploying to a production environment, all TODOs must be completed.

### Findings

- [src/vault/adapter/abstracts/AdapterBase.sol#L516](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L516)
  ```Solidity
  // TODO use deterministic fee recipient proxy
  ```

### References

- [TODO comments in the code base | zkSync Layer 1 Audit](https://blog.openzeppelin.com/zksync-layer-1-audit/#todo-comments-in-the-code-base)


## [L-02] Use of abi.encodePacked instead of bytes.concat() for Solidity version >= 0.8.4

### Description

From the Solidity version `0.8.4` it was added the possibility to use `bytes.concat` with variable number of `bytes` and `bytesNN` arguments. With a more evocative name, it functions as a restricted `abi.encodePacked`.

### Findings

- [src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol) => `pragma solidity ^0.8.15;`
  ```Solidity
  ::104 =>     bytes32 id = keccak256(abi.encodePacked(token, account, amount, nonce));
  ```
- [src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol) => `pragma solidity ^0.8.15;`
  ```Solidity
  ::49 =>     _name = string(abi.encodePacked("Staked ", IERC20Metadata(address(_stakingToken)).name()));
  ::50 =>     _symbol = string(abi.encodePacked("pst-", IERC20Metadata(address(_stakingToken)).symbol()));
  ::461 =>           abi.encodePacked(
  ```
- [src/vault/Vault.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol) => `pragma solidity ^0.8.15;`
  ```Solidity
  ::94 =>             abi.encodePacked("Popcorn", name(), block.timestamp, "Vault")
  ::680 =>                     abi.encodePacked(
  ```
- [src/vault/adapter/abstracts/AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol) => `pragma solidity ^0.8.15;`
  ```Solidity
  ::648 =>                     abi.encodePacked(
  ```

### References

- [Solidity 0.8.4 Release Announcement](https://blog.soliditylang.org/2021/04/21/solidity-0.8.4-release-announcement/)
- [Remove abi.encodePacked #11593](https://github.com/ethereum/solidity/issues/11593)


## [L-03] Unnamed return parameters

### Description

To increase explicitness and readability, take into account introducing and utilizing named return parameters.

### Findings

- [src/utils/EIP165.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/EIP165.sol)
  ```Solidity
  ::7 =>   function supportsInterface(bytes4 interfaceId) public view virtual returns (bool) {}
  ```
- [src/utils/MultiRewardEscrow.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol)
  ```Solidity
  ::38 =>   function getEscrowIdsByUser(address account) external view returns (bytes32[] memory) {
  ::42 =>   function getEscrowIdsByUserAndToken(address account, IERC20 token) external view returns (bytes32[] memory) {
  ::51 =>   function getEscrows(bytes32[] calldata escrowIds) external view returns (Escrow[] memory) {
  ::140 =>   function isClaimable(bytes32 escrowId) external view returns (bool) {
  ::144 =>   function getClaimableAmount(bytes32 escrowId) external view returns (uint256) {
  ::170 =>   function _getClaimableAmount(Escrow memory escrow) internal view returns (uint256) {
  ```
- [src/utils/MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol)
  ```Solidity
  ::59 =>   function name() public view override(ERC20Upgradeable, IERC20Metadata) returns (string memory) {
  ::63 =>   function symbol() public view override(ERC20Upgradeable, IERC20Metadata) returns (string memory) {
  ::67 =>   function decimals() public view override(ERC20Upgradeable, IERC20Metadata) returns (uint8) {
  ::75 =>   function deposit(uint256 _amount) external returns (uint256) {
  ::79 =>   function mint(uint256 _amount) external returns (uint256) {
  ::83 =>   function withdraw(uint256 _amount) external returns (uint256) {
  ::87 =>   function redeem(uint256 _amount) external returns (uint256) {
  ::98 =>   function _convertToShares(uint256 assets, Math.Rounding) internal pure override returns (uint256) {
  ::102 =>   function _convertToAssets(uint256 shares, Math.Rounding) internal pure override returns (uint256) {
  ::362 =>   function getAllRewardsTokens() external view returns (IERC20[] memory) {
  ::487 =>   function DOMAIN_SEPARATOR() public view returns (bytes32) {
  ::491 =>   function computeDomainSeparator() internal view virtual returns (bytes32) {
  ```
- [src/vault/VaultController.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol)
  ```Solidity
  ::278 =>   function deployStaking(IERC20 asset) external canCreate returns (address) {
  ```
- [src/vault/CloneRegistry.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol)
  ```Solidity
  ::65 =>   function getAllClones() external view returns (address[] memory) {
  ```
- [src/vault/PermissionRegistry.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main//blob/main/src/vault/PermissionRegistry.sol)
  ```Solidity
  ::51 =>   function endorsed(address target) external view returns (bool) {
  ::55 =>   function rejected(address target) external view returns (bool) {
  ```
- [src/vault/TemplateRegistry.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol)
  ```Solidity
  ::115 =>   function getTemplateCategories() external view returns (bytes32[] memory) {
  ::119 =>   function getTemplateIds(bytes32 templateCategory) external view returns (bytes32[] memory) {
  ::123 =>   function getTemplate(bytes32 templateCategory, bytes32 templateId) external view returns (Template memory) {
  ```
- [src/vault/Vault.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)
  ```Solidity
  ::100 =>     function decimals() public view override returns (uint8) {
  ::124 =>     function deposit(uint256 assets) public returns (uint256) {
  ::160 =>     function mint(uint256 shares) external returns (uint256) {
  ::200 =>     function withdraw(uint256 assets) public returns (uint256) {
  ::242 =>     function redeem(uint256 shares) external returns (uint256) {
  ::285 =>     function totalAssets() public view returns (uint256) {
  ::294 =>     function convertToShares(uint256 assets) public view returns (uint256) {
  ::308 =>     function convertToAssets(uint256 shares) public view returns (uint256) {
  ::399 =>     function maxDeposit(address caller) public view returns (uint256) {
  ::404 =>     function maxMint(address caller) external view returns (uint256) {
  ::409 =>     function maxWithdraw(address caller) external view returns (uint256) {
  ::414 =>     function maxRedeem(address caller) external view returns (uint256) {
  ::429 =>     function accruedManagementFee() public view returns (uint256) {
  ::447 =>     function accruedPerformanceFee() public view returns (uint256) {
  ::709 =>     function DOMAIN_SEPARATOR() public view returns (bytes32) {
  ::716 =>     function computeDomainSeparator() internal view virtual returns (bytes32) {
  ```
- [src/vault/VaultRegistry.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultRegistry.sol)
  ```Solidity
  ::59 =>   function getVault(address vault) external view returns (VaultMetadata memory) {
  ::63 =>   function getVaultsByAsset(address asset) external view returns (address[] memory) {
  ::67 =>   function getTotalVaults() external view returns (uint256) {
  ::71 =>   function getRegisteredAddresses() external view returns (address[] memory) {
  ::75 =>   function getSubmitter(address vault) external view returns (VaultMetadata memory) {
  ```
- [src/vault/adapter/abstracts/AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol)
  ```Solidity
  ::247 =>     function totalAssets() public view override returns (uint256) {
  ::258 =>     function _totalAssets() internal view virtual returns (uint256) {}
  ::265 =>     function _underlyingBalance() internal view virtual returns (uint256) {}
  ::419 =>     function maxMint(address) public view virtual override returns (uint256) {
  ::529 =>     function accruedPerformanceFee() public view returns (uint256) {
  ::677 =>     function DOMAIN_SEPARATOR() public view virtual returns (bytes32) {
  ::684 =>     function computeDomainSeparator() internal view virtual returns (bytes32) {
  ```
- [src/vault/adapter/abstracts/WithRewards.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/WithRewards.sol)
  ```Solidity
  ::13 =>   function rewardTokens() external view virtual returns (address[] memory) {}
  ::21 =>   function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {
  ```
- [src/vault/adapter/beefy/BeefyAdapter.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol)
  ```Solidity
  ::108 =>     function _totalAssets() internal view override returns (uint256) {
  ::117 =>     function _underlyingBalance() internal view override returns (uint256) {
  ::136 =>     function rewardTokens() external view override returns (address[] memory) {
  ```
- [src/vault/adapter/yearn/YearnAdapter.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol)
  ```Solidity
  ::80 =>     function _totalAssets() internal view override returns (uint256) {
  ::84 =>     function _underlyingBalance() internal view override returns (uint256) {
  ::89 =>     function _shareValue(uint256 yShares) internal view returns (uint256) {
  ::101 =>     function _freeFunds() internal view returns (uint256) {
  ::109 =>     function _yTotalAssets() internal view returns (uint256) {
  ::114 =>     function _calculateLockedProfit() internal view returns (uint256) {
  ::144 =>     function maxDeposit(address) public view override returns (uint256) {
  ```

### References

- [Unnamed return parameters | Opyn Bull Strategy Contracts Audit](https://blog.openzeppelin.com/opyn-bull-strategy-contracts-audit/#unnamed-return-parameters)