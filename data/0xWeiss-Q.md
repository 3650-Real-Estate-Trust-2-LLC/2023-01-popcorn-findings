# [L-01] CHANGE VARIBALES TO IMMUTABLE

The following variables, do not have a function to update them and are initialized in the constructor, therefore they should be immutable or constant and hardcode the addresses.

    IBeefyVault public beefyVault;
    IBeefyBooster public beefyBooster;
    IBeefyBalanceCheck public beefyBalanceCheck;

The variables are here:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L27-L29

# [L-02] UNUSAGE OF ECDSA LIBRARY AND WRONGLY CHECKED THE RECOVERED ADDRESS

The issue relies on the permit function. Where Popcorn is using the recover opcode instead of the ECDSA library from openzeppelin. That does not bring any issues normally, but it is the best practice. Also, due to not using the ECDSA library,  
the function requirement can be bypassed

    function permit(
        address owner,
        address spender,
        uint256 value,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) public virtual {
        if (deadline < block.timestamp) revert PermitDeadlineExpired(deadline);

        // Unchecked because the only math done is incrementing
        // the owner's nonce which cannot realistically overflow.
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
                                owner,
                                spender,
                                value,
                                nonces[owner]++,
                                deadline
                            )
                        )
                    )
                ),
                v,
                r,
                s
            );
            if (recoveredAddress == address(0) || recoveredAddress != owner)
                revert InvalidSigner(recoveredAddress);

            _approve(recoveredAddress, spender, value);
        }

As you can notice, the if statement:

       if (recoveredAddress == address(0) || recoveredAddress != owner)

it is not truly well implemented, because in the function it is not checked that owner != address(0). So, if you pass the owner as address(0) and random signature values. The recovered address will be 0. So it will bypass the check

        (recoveredAddress == address(0) || recoveredAddress != owner)

because it has and OR || not and  &&.

## Recommendation

Change the way in how you formulate that if statement:

         if (recoveredAddress != address(0) && recoveredAddress == owner)

and replace recover for the ECDSA library by OZ