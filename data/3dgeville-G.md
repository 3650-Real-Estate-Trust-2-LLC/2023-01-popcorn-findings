Line:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneFactory.sol#L39

function deploy(Template calldata template, bytes calldata data)

Use bytes32 instead of bytes. Dynamic arrays cost more.
