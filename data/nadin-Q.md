## [N-01] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)
Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled
Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,).
Context: 06
File: src/utils/MultiRewardEscrow.sol

    104:    bytes32 id = keccak256(abi.encodePacked(token, account, amount, nonce));

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L104

File: src/utils/MultiRewardStaking.sol 

    49:    _name = string(abi.encodePacked("Staked ", IERC20Metadata(address(_stakingToken)).name()));
    50:    _symbol = string(abi.encodePacked("pst-", IERC20Metadata(address(_stakingToken)).symbol()));
    461:          abi.encodePacked(

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L49-L50
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L461

File: src/vault/Vault.sol

    94:            abi.encodePacked("Popcorn", name(), block.timestamp, "Vault")
    680:                    abi.encodePacked(

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L94
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L680

File: src/vault/adapter/abstracts/AdapterBase.sol

    648:                     abi.encodePacked(

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L648

## [N-02] Using ecrecover is against best practice
Context: 03
Using ecrecover is against best practice. Preferably use ECDSA.recover instead. EIP-2 still allows signature malleability for ecrecover(). Remove this possibility and make the signature unique. However it should be impossible to be a threat by now.
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v3.4.0/contracts/cryptography/ECDSA.sol

File: src/utils/MultiRewardStaking.sol 

    459:      address recoveredAddress = ecrecover(

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L459-L479

File: src/vault/Vault.sol

    678:            address recoveredAddress = ecrecover(

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L678

File: src/vault/adapter/abstracts/AdapterBase.sol

    646:             address recoveredAddress = ecrecover(

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L646

## [N-03] NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES
All contract

## [NC-04] Return values of approve() not checked
Not all IERC20 implementations revert() when there's a failure in approve(). The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything
File: src/utils/MultiRewardStaking.sol , src/vault/Vault.sol , src/vault/VaultController.sol
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol

## [N-05] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS
Context:
All Contracts

Description:
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

Recommendation:
NatSpec comments should be increased in contracts