# Lack of Input Validation

For defense-in-depth purposes, it is recommended to perform additional validation against the amount the user is attempting to deposit, mint, withdraw and redeem to ensure that the submitted amount is valid.

[OpenZeppelinTokenizedVault.sol#L95](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/7c75b8aa89073376fb67d78a40f6d69331092c94/contracts/token/ERC20/extensions/ERC20TokenizedVault.sol#L95)

## 1. Function `deposit()`


[Vault.sol#L134-L158](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L134-L158)

```Solidity
 function deposit(uint256 assets, address receiver)
        public
        nonReentrant
        whenNotPaused
        syncFeeCheckpoint
        returns (uint256 shares)
    {
       require(assets <= maxDeposit(receiver), "deposit more than max");
```

## 2. Function `mint()`

[Vault.sol#L170-L198](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L170-L198)

```Solidity
 function mint(uint256 shares, address receiver)
        public
        nonReentrant
        whenNotPaused
        syncFeeCheckpoint
        returns (uint256 assets)
    {
        require(shares <= maxMint(receiver), "mint more than max");
```

## 3. Function `withdraw()`

[Vault.sol#L211-L240](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L211-L240)

```Solidity
 function withdraw(
        uint256 assets,
        address receiver,
        address owner
    ) public nonReentrant syncFeeCheckpoint returns (uint256 shares) {
        require(assets <= maxWithdraw(owner), "withdraw more than max");
```

## 4. Function `redeem()`

[Vault.sol#L253-L278](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L253-L278)

```Solidity
function redeem(
        uint256 shares,
        address receiver,
        address owner
    ) public nonReentrant returns (uint256 assets) {
        require(shares <= maxRedeem(owner), "redeem more than max");
```

## 5. Function `deposit()`


[AdapterBase.sol#L110-L122](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L110-L122)

```Solidity
 function deposit(uint256 assets, address receiver)
        public
        virtual
        override
        returns (uint256)
    {
       require(assets <= maxDeposit(receiver), "deposit more than max");
```

## 6. Function `mint()`

[AdapterBase.sol#L129-L141](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L129-L141)

```Solidity
 function mint(uint256 shares, address receiver)
        public
        virtual
        override
        returns (uint256)
    {
        require(shares <= maxMint(receiver), "mint more than max");
```

## 7. Function `withdraw()`

[AdapterBase.sol#L173-L185](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L173-L185)

```Solidity
function withdraw(
        uint256 assets,
        address receiver,
        address owner
    ) public virtual override returns (uint256) {
        require(assets <= maxWithdraw(owner), "withdraw more than max");
```

## 8. Function `redeem()`

[AdapterBase.sol#L193-L204](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L193-L204)

```Solidity
function redeem(
        uint256 shares,
        address receiver,
        address owner
    ) public virtual override returns (uint256) {
        require(shares <= maxRedeem(owner), "redeem more than max");
```

<br>

# Critical Address Changes Should Use Two-Step Procedure

## Context

[Vault.sol#L553-L559](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L553-L559)

```Solidity
function setFeeRecipient(address _feeRecipient) external onlyOwner {
        if (_feeRecipient == address(0)) revert InvalidFeeRecipient();

        emit FeeRecipientUpdated(feeRecipient, _feeRecipient);

        feeRecipient = _feeRecipient;
    }
```

## Recommendation

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.