## Missing parameter validation in constructor
Some parameters of constructors are not checked for invalid values.
[MultiRewardEscrow.sol#L30-L32](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L30-L32)
[DeploymentController.sol#L35-L44](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L35-L44)
[VaultController.sol#L53-L70](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L53-L70)

## Use of ecrecover is susceptible to signature malleability
The built-in EVM precompile ecrecover is susceptible to signature malleability, which could lead to replay attacks. References: https://swcregistry.io/docs/SWC-117, https://swcregistry.io/docs/SWC-121, and https://medium.com/cryptronics/signature-replay-vulnerabilities-in-smart-contracts-3b6f7596df57. While this is not immediately exploitable, this may become a vulnerability if used elsewhere.   
```
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
    }
```
[MultiRewardStaking.sol#L459](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L459)
[Vault.sol#L678](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L678)
[AdapterBase.sol#L646](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L646)

## Use `safeTransfer/safeTransferFrom` consistently instead of transfer/transferFrom
Some tokens do not revert on failure, but instead return false so its better to use `safeTransfer/safeTransferFrom` and same for  `safeapprove`
```
IERC20(rewardsToken).transferFrom(msg.sender, address(adminProxy), amount);
```
[VaultController.sol#L456](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L456)
[VaultController.sol#L457](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L457)

## Missing address(0x0) check
function should check for 0 address check
[CloneRegistry.sol#L41-L51](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneRegistry.sol#L41-L51)
[MultiRewardStaking.sol#L170](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L170)
[DeploymentController.sol#L121-L125](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L121-L125)
[PermissionRegistry.sol#L38-L49](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L38-L49)
[Vault.sol#L211-L240](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L211-L240)
[]()
[]()
[]()
[]()
[]()
[]()
[]()
