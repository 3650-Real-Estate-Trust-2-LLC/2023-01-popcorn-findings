There are functions for pause and unpause in the vault contract, but i don't see anywhere that contract is using them!

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L643
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L648

if you don't want use them, remove them to save gas.

////////////////////////////////////////////// ***** //////////////////////////////////////////////

in the below functions, only the owner can access them and make changes in contract data.

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L525
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L553
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L578
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L643
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L648

if it's possible change this to The File Pattern, 

function file(bytes32 what, bytes calldata value) external onlyAdmin {
  if (what == "token") token = abi.decode(value, (address));
  else if (what == "min") min = abi.decode(value, (uint256));
  else if (what == "max") max = abi.decode(value, (uint256));
  else if (what == "hash") hash = abi.decode(value, (bytes32));
  else revert InvalidParameter(what);
  emit File(what, value);
}

it's just a big if statement. We check for each supported what key to determine which value to set, and dynamically ABI-decode the value bytes into an address, uint256, or bytes32 when we find a match.

In addition to being more concise, file has a few other advantages:

The onlyAdmin modifier is applied just once, to a single authenticated function. It's easy to miss or delete one of these modifiers when creating multiple setter functions.

It's possible to monitor the File event for all configuration changes, and reconstruct the configuration history of the contract from one event rather than collating multiple different events.

source :
https://mirror.xyz/horsefacts.eth/R5N_Dzm2XKVvSI3cM8e8EHuFzvpPBttG2TEtyEZDa10

////////////////////////////////////////////// ***** //////////////////////////////////////////////

in the below function, you are to Simulate the effects of a deposit at the current block, given current on-chain conditions.
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L323

this function is not used in the vault contract and only reads data from the adapter contract. so why you don't read this function directly from the adapter contract in the web3.0 section or from the front-end and remove this function from the vault contract?

////////////////////////////////////////////// ***** //////////////////////////////////////////////

++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops
The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop.

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L806
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L766
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L646
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L633
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L620
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L607
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L587
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L564
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L523
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L494