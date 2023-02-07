## Summary

### Low Risk Issues

|  | Issue | Contexts |
| --- | --- | --- |
| LOW‑1 | Use _safeMint instead of _mint | 6 |
| LOW‑2 | Lock pragmas to specific compiler version | All Contracts |

### Non-critical Issues

|  | Issue | Contexts |
| --- | --- | --- |
| NC‑1 |  Public functions not called by the contract should be declared external instead | 4 |
| NC‑2 | Add parameter to Event-Emit | 1 |
| NC‑3 | Include return parameters in NatSpec comments | All contracts |
| NC‑4 | NatSpec is missing | 3 |

### [L‑1] Use `_safeMint` instead of `_mint`

According to openzepplin's ERC721, the use of `_mint` is discouraged, use `_safeMint` whenever possible. [https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-_mint-address-uint256-](https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-_mint-address-uint256-)

```jsx
/src/vault/adapter/abstracts/AdapterBase.sol
160: _mint(receiver, shares);

/src/vault/Vault.sol
151: _mint(receiver, shares);
191: _mint(receiver, shares);

/src/utils/MultiRewardStaking.sol
80:  mint(_amount, msg.sender);
115: _mint(receiver, shares);
148: _mint(to, amount);

```

### [L-02] *Lock pragmas* to specific compiler version

**Description:** Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally. [https://swcregistry.io/docs/SWC-103](https://swcregistry.io/docs/SWC-103)

**Recommendation:** Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version. [solidity-specific/locking-pragmas](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)

```jsx
All Contracts
pragma solidity ^0.8.15;
```

### [N-01] Public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents' functions and change the visibility from external to public. Functions marked by external use call data to read arguments, where public will first allocate in local memory and then load them.

```jsx
/src/vault/adapter/abstracts/AdapterBase.sol
271: function convertToUnderlyingShares(uint256 assets, uint256 shares)
456: function strategyDeposit(uint256 amount, uint256 shares)
467: function strategyWithdraw(uint256 amount, uint256 shares)
610: function supportsInterface(bytes4 interfaceId)
```

### [NC-02] Add parameter to Event-Emit

Some event-emit description hasn’t parameter. Add to parameter for front-end website or client app , they can has that something has happened on the blockchain.

```jsx
/src/vault/adapter/abstracts/AdapterBase.sol
449: emit Harvested();
```

### **[NC-03] Include `return parameters` in *NatSpec comments***

**Context:** All Contracts

**Description:**

[https://docs.soliditylang.org/en/v0.8.15/natspec-format.html](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html)

If Return parameters are declared, you must prefix them with "/// @return".

Some code analysis programs do analysis by reading NatSpec details, if they can't see the `@return` tag, they do incomplete analysis.

**Recommendation:** Include return parameters in NatSpec comments

Recommendation Code Style:

```jsx
   /// @notice information about what a function does
   /// @param pageId The id of the page to get the URI for.
   /// @return Returns a page's URI if it has been minted 
   function tokenURI(uint256 pageId) public view virtual override returns (string memory) {
       if (pageId == 0 || pageId > currentId) revert("NOT_MINTED");

       return string.concat(BASE_URI, pageId.toString());
   }
```

### [NC-04] NatSpec is missing

**Description:** NatSpec is missing for the following functions , constructor and modifier:

```jsx
/src/vault/VaultController.sol
704: modifier canCreate() {

/src/vault/Vault.sol#
496: modifier syncFeeCheckpoint() {

/src/utils/MultiRewardStaking.sol
362: function getAllRewardsTokens() external view returns (IERC20[] memory) {
```