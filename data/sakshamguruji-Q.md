## USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)

### Description:

Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled

Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,).

Affected instances:

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L104
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L461
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L680
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L648

## EVENT MISORDERING POSSIBLITY

### Description:

The withdrawTo function of the StakeManager contract emits an event after sending the funds. This violates the
Checks-Effects-Interactions pattern and it introduces the possibility that some events could be emitted out of order
if the recipient’s fallback function executes another operation. Consider emitting the event before the funds transfer.

Affected instances:

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L166
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L117
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L133
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L150
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L180
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L183
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L158
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L197
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L239
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L277
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L164
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L234


## ALWAYS CHECK IF OPENZEPPELIN CONTRACTS ARE UP TO DATE

### Description:

The openzeppelin version used is 4.8.0 whereas the latest version is 4.8.1 , it should always be ensured that latest version is used to avoid errors/bugs
in the previous versions.

## FUNCTIONS, PARAMETERS AND VARIABLES IN SNAKE CASE

### Description:

Use camel case for all functions, parameters and variables and snake case for constants

Affected instances:

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L35
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L40
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L55

## VARIABLE NAME SHOULD BE DAYS PER YEAR INSTEAD 

### Description:

Here https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L35 the name of the variable should be 
days per year instead of seconds per year as the value is denoted in days.

## MISSING DOCSTRINGS

### Description:

Many functions in the code base lack documentation. This hinders reviewers’ understanding of the code’s intention,
which is fundamental to correctly assess not only security, but also correctness. Additionally, docstrings improve
readability and ease maintenance. They should explicitly explain the purpose or intention of the functions,
the scenarios under which they can fail, the roles allowed to call them, the values returned and the events emitted.

Consider thoroughly documenting all functions (and their parameters) that are part of the contracts’ public API.
Functions implementing sensitive functionality, even if not public, should be clearly documented as well.
When writing docstrings, consider following the Ethereum Natural Specification Format (NatSpec).

Example - https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L88 Here 
lacks info about returned values

## COMMENT SAYS "DAYS" INSTEAD OF SECONDS

### Description:

Here https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L501 the comment should be 
"Dont wait more than X days" instead.



