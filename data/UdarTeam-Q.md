# [Q-01] Wrong NatSpec Documentation

src/vault/VaultController.sol: [L79](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L79) Says that "Caller must be owner", but the [modifier](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L97) on the function is not the ```onlyOwner``` modifier, instead it is ```canCreate``` and it does not check if ```msg.sender == owner```.
