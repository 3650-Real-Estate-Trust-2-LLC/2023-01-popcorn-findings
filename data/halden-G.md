# [01-G] Remove redundant events
Remove redundant events will decrease deployment gas.
File TemplateRegistry.sol: line [40](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L40)

# [G-02] Add unchecked {} where the operands can not underflow/overflow
File Vault.sol line [541](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L541) 
File MultiRewardEscrow.sol:
line 162: `escrows[escrowId].balance -= claimable;`  can not underflow because function `_getClaimableAmount` can return maximum value equal to escrow.balance line 110: `amount -= fee;` , if the feePerc is equal to 1e18, the maximum value for the calculated fee it will be equal to amount.
File MultiRewardStaking.sol:
line 198: `uint256 payout = rewardAmount - escrowed;`
File AdapterBase.sol
line 158: `underlyingBalance += _underlyingBalance() - underlyingBalance_;`

# [G-03] Minimize SLOAD opcode
1. Variable `proposedFees` should be cached in memory to minimize SLOAD opcode
File Vault.sol line [544](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L544)
Also in this function should be cached `quitPeriod`
Recommended:
```
VaultFее memory newFee = proposedFees;
emit ChangedFees(fees, newFee);
fees = newFee;
```
2. Use argument variable `_quitPeriod` instead of `quitPeriod` for emmiting event to minimize SLOAD opcode
File Vault.sol lines [635](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L635)
Recommended:
`emit QuitPeriodSet(_quitPeriod);`

3. Variable `owner` should be cached to minimize SLOAD opcode.
File Vault.sol lins [664-705](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L664-L705) and also in AdapterBase.sol lines [632-673](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L632-L673)
The variable is used three times in the function.
Recommended:
```
address contractOwner = owner;
unchecked {
            address recoveredAddress = ecrecover(
                keccak256(
                    abi.encodePacked(
                        "\x19\x01",
                        DOMAIN_SEPARATOR(),
                        keccak256(
                            abi.encode(
                                keccak256(
                                    "Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"
                                ),
                                contractOwner, // save SLOAD
                                spender,
                                value,
                                nonces[contractOwner]++, // save SLOAD
                                deadline
                            )
                        )
                    )
                ),
                v,
                r,
                s
            );

            if (recoveredAddress == address(0) || recoveredAddress != contractOwner) // save SLOAD
``` 
4. Variable `owner` should be cached to minimize SLOAD opcode.
File MultiRewardStaking.sol:
line [128-133](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L128-L133)
Recommended:
```
address contractOwner = owner;
if (caller != contractOwner) _approve(contractOwner, msg.sender, allowance(contractOwner, msg.sender) - shares);

_burn(contractOwner, shares);
IERC20(asset()).safeTransfer(receiver, assets);

emit Withdraw(caller, receiver, contractOwner, assets, shares);
```

5. Variable `owner` should be cached to minimize SLOAD opcode.
AdapterBase.sol lines [217-234](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L217-L234)

# [G-04] Using unchecked blocks to save gas - Increments in for loop can be unchecked
File MultiRewardEscrow.sol:
line 53: `for (uint256 i = 0; i < escrowIds.length; i++) `
line 155: `for (uint256 i = 0; i < escrowIds.length; i++)`
line 210: `for (uint256 i = 0; i < tokens.length; i++)`
File MultiRewardStaking.sol:
line 373: `for (uint8 i; i < _rewardTokens.length; i++)`
line 172: `for (uint8 i; i < _rewardTokens.length; i++)`

# [G-05] EIP712 Domain can be constant
the current EIP712 Domain is not expected to be changed so it can be constant variable.
File MultiRewardStaking.sol: line [495](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L495)
Also in AdapterBase.sol line [689](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L689)
### Recommended:
bytes32 constant EIP712DOMAIN_HASH = ` keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)")`

```
keccak256(
        abi.encode(
          EIP712DOMAIN_HASH
          keccak256(bytes(name())),
          keccak256("1"),
          block.chainid,
          address(this)
        )
      );
```
Same applied for `"Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"`

# [G-06] Mark constructor as payable to save gas
Constructors can be marked as payable to save gas.