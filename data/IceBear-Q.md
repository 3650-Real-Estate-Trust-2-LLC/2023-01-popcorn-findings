## [L-01]Use of floating pragma 
### Description
https://swcregistry.io/docs/SWC-103

### Context
All Contracts.


### Recommended Mitigation Steps
Use fixed solidity version.


## [L-02] Use of Block.timestamp
### Description
Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.
### Context
All Contracts.

## [N-01] INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS
### Context
All Contracts.

### Description

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

If Return parameters are declared, you must prefix them with ”/// @return”.

Some code analysis programs do analysis by reading NatSpec details, if they can’t see the “@return” tag, they do incomplete analysis.

### Recommended Mitigation Steps
Include return parameters in NatSpec comments

Recommendation Code Style:
```
		/// @notice information about what a function does
		/// @param pageId The id of the page to get the URI for.
		/// @return Returns a page's URI if it has been minted 
		function tokenURI(uint256 pageId) public view virtual override returns (string memory) {
				if (pageId == 0 || pageId > currentId) revert("NOT_MINTED");
				return string.concat(BASE_URI, pageId.toString());
		}
```

## [N-02] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier

### Context
[src/vault/Vault.sol#L35](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L35
)
```solidity=
uint256 constant SECONDS_PER_YEAR = 365.25 days;
```

## [N-03] SIGNATURE MALLEABILITY OF EVM’S ECRECOVER()

### Context
[src/utils/MultiRewardStaking.sol#L459](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L459)
[src/vault/Vault.sol#L678](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol)
[src/vault/adapter/abstracts/AdapterBase.sol#L646](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L646)

### Description: 
The function calls the Solidity ecrecover() function directly to verify the given signatures. However, the ecrecover() EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

Although a replay attack seems not possible for this contract, I recommend using the battle-tested OpenZeppelin’s ECDSA library.

### Recommended Mitigation Steps
Use the ecrecover function from OpenZeppelin’s ECDSA library for signature verification. (Ensure using a version > 4.7.3 for there was a critical bug >= 4.1.0 < 4.7.3).

## [N-04] EMPTY BLOCKS SHOULD BE REMOVED OR EMIT SOMETHING

### Description: 
Code contains empty block.

### Recommended Mitigation Steps：
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.