# [Q-01] Wrong NatSpec Documentation

src/vault/VaultController.sol: [L79](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L79) Says that "Caller must be owner", but the [modifier](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L97) on the function is not the ```onlyOwner``` modifier, instead it is ```canCreate``` and it does not check if ```msg.sender == owner```.

# [Q-02] Mark abstarct contracts as abstract

Contracts [OnlyStrategy](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/OnlyStrategy.sol) and [WithRewards](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/WithRewards.sol) are meant to be abstract, but not marked like that.

```diff
-contract OnlyStrategy {
+abstract contract  OnlyStrategy {

-contract WithRewards is EIP165, OnlyStrategy {
+abstract contract WithRewards is EIP165, OnlyStrategy {
```

# [Q-03] Return the escrow id to user immediately after creating one, for better user experience.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L88

```diff
-  ) external {
+  ) external returns(bytes32 id){
     if (token == IERC20(address(0))) revert ZeroAddress();
     if (account == address(0)) revert ZeroAddress();
     if (amount == 0) revert ZeroAmount();
@@ -101,7 +101,7 @@ contract MultiRewardEscrow is Owned {
 
     nonce++;
 
-    bytes32 id = keccak256(abi.encodePacked(token, account, amount, nonce));
+    id = keccak256(abi.encodePacked(token, account, amount, nonce));
 
     uint256 feePerc = fees[token].feePerc;
     if (feePerc > 0) {
```
