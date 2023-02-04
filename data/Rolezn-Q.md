## Summary<a name="Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW&#x2011;1) | Use of `ecrecover` is susceptible to signature malleability | 3 |
| [LOW&#x2011;2](#LOW&#x2011;2) | `decimals()` not part of ERC20 standard | 4 |
| [LOW&#x2011;3](#LOW&#x2011;3) | The Contract Should `approve(0)` First | 8 |
| [LOW&#x2011;4](#LOW&#x2011;4) | Missing parameter validation in `constructor` | 2 |
| [LOW&#x2011;5](#LOW&#x2011;5) | Missing Contract-existence Checks Before Low-level Calls | 2 |
| [LOW&#x2011;6](#LOW&#x2011;6) | Contracts are not using their OZ Upgradeable counterparts | 3 |
| [LOW&#x2011;7](#LOW&#x2011;7) | Unbounded loop | 9 |
| [LOW&#x2011;8](#LOW&#x2011;8) | No Storage Gap For Upgradeable Contracts | 2 |
| [LOW&#x2011;9](#LOW&#x2011;9) | Upgrade OpenZeppelin Contract Dependency | 1 |
| [LOW&#x2011;10](#LOW&#x2011;10) | Missing `delegatecall` success check | 1 |
| [LOW&#x2011;11](#LOW&#x2011;11) | `SECONDS_PER_YEAR` value is incorrect  | 1 |

Total: 36 contexts over 11 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC&#x2011;1) | Add a timelock to critical functions | 9 |
| [NC&#x2011;2](#NC&#x2011;2) | Avoid Floating Pragmas: The Version Should Be Locked | 35 |
| [NC&#x2011;3](#NC&#x2011;3) | Variable Names That Consist Of All Capital Letters Should Be Reserved For Const/immutable Variables | 1 |
| [NC&#x2011;4](#NC&#x2011;4) | Critical Changes Should Use Two-step Procedure | 9 |
| [NC&#x2011;5](#NC&#x2011;5) | `block.timestamp` is already used when emitting events, no need to input timestamp | 2 |
| [NC&#x2011;6](#NC&#x2011;6) | Event emit should emit a parameter | 1 |
| [NC&#x2011;7](#NC&#x2011;7) | Function writing that does not comply with the Solidity Style Guide  | 35 |
| [NC&#x2011;8](#NC&#x2011;8) | Use `delete` to Clear Variables | 2 |
| [NC&#x2011;9](#NC&#x2011;9) | Imports can be grouped together | 44 |
| [NC&#x2011;10](#NC&#x2011;10) | NatSpec return parameters should be included in contracts | All in-scope contracts |
| [NC&#x2011;11](#NC&#x2011;11) | Insufficient coverage | 1 |
| [NC&#x2011;12](#NC&#x2011;12) | Lines are too long | 1 |
| [NC&#x2011;13](#NC&#x2011;13) | Missing event for critical parameter change | 8 |
| [NC&#x2011;14](#NC&#x2011;14) | Implementation contract may not be initialized | 9 |
| [NC&#x2011;15](#NC&#x2011;15) | NatSpec comments should be increased in contracts | All in-scope contracts |
| [NC&#x2011;16](#NC&#x2011;16) | Use a more recent version of Solidity | 35 |
| [NC&#x2011;17](#NC&#x2011;17) | Open TODOs | 1 |
| [NC&#x2011;18](#NC&#x2011;18) | Use `bytes.concat()` | 1 |
| [NC&#x2011;19](#NC&#x2011;19) | Use of Block.Timestamp | 8 |

Total: 281 contexts over 19 issues

## Low Risk Issues




### <a href="#Summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Use of `ecrecover` is susceptible to signature malleability

The built-in EVM precompile `ecrecover` is susceptible to signature malleability, which could lead to replay attacks.
References:  https://swcregistry.io/docs/SWC-117,  https://swcregistry.io/docs/SWC-121, and  https://medium.com/cryptronics/signature-replay-vulnerabilities-in-smart-contracts-3b6f7596df57.
While this is not immediately exploitable, this may become a vulnerability if used elsewhere.

#### <ins>Proof Of Concept</ins>


```solidity
459: function permit(
    address owner,
    address spender,
    uint256 value,
    uint256 deadline,
    uint8 v,
    bytes32 r,
    bytes32 s
  ) public virtual {
    if (deadline < block.timestamp) revert PermitDeadlineExpired(deadline);

    
    
    unchecked {
      address recoveredAddress = ecrecover(
        keccak256(
          abi.encodePacked(
            "/x19/x01",
            DOMAIN_SEPARATOR(),
            keccak256(
              abi.encode(
                keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),
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

      if (recoveredAddress == address(0) || recoveredAddress != owner) revert InvalidSigner(recoveredAddress);

      _approve(recoveredAddress, spender, value);
    }
  }
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L459

```solidity
678: function permit(
        address owner,
        address spender,
        uint256 value,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) public virtual {
        if (deadline < block.timestamp) revert PermitDeadlineExpired(deadline);

        
        
        unchecked {
            address recoveredAddress = ecrecover(
                keccak256(
                    abi.encodePacked(
                        "/x19/x01",
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

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L678

```solidity
646: function permit(
        address owner,
        address spender,
        uint256 value,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) public virtual {
        if (deadline < block.timestamp) revert PermitDeadlineExpired(deadline);

        
        
        unchecked {
            address recoveredAddress = ecrecover(
                keccak256(
                    abi.encodePacked(
                        "/x19/x01",
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

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L646



#### Recommended Mitigation Steps
Consider using OpenZeppelin’s ECDSA library (which prevents this malleability) instead of the built-in function.
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol#L138-L149



### <a href="#Summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> `decimals()` not part of ERC20 standard

`decimals()` is not part of the official ERC20 standard and might fail for tokens that do not implement it. While in practice it is very unlikely, as usually most of the tokens implement it, this should still be considered as a potential issue.

#### <ins>Proof Of Concept</ins>


```solidity
51: _decimals = IERC20Metadata(address(_stakingToken)).decimals()
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L51

```solidity
274: uint64 ONE = (10**IERC20Metadata(address(rewardToken)).decimals()
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L274

```solidity
82: _decimals = IERC20Metadata(address(asset_)).decimals()
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L82

```solidity
77: _decimals = IERC20Metadata(asset).decimals()
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L77










### <a href="#Summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> The Contract Should `approve(0)` First

Some tokens (like USDT L199) do not work when changing the allowance from an existing non-zero allowance value. They must first be approved by zero and then the actual allowance must be approved.

#### <ins>Proof Of Concept</ins>


```solidity
80: asset.approve(address(adapter_), type(uint256).max);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L80

```solidity
604: asset.approve(address(adapter), 0);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L604

```solidity
610: asset.approve(address(adapter), type(uint256).max);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L610

```solidity
171: asset.approve(address(target), initialDeposit);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L171

```solidity
456: IERC20(rewardsToken).approve(staking, type(uint256).max);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L456

```solidity
80: IERC20(asset()).approve(_beefyVault, type(uint256).max);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L80

```solidity
83: IERC20(_beefyVault).approve(_beefyBooster, type(uint256).max);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L83

```solidity
54: IERC20(_asset).approve(address(yVault), type(uint256).max);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L54



#### <ins>Recommended Mitigation Steps</ins>

Approve with a zero amount first before setting the actual amount.



### <a href="#Summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> Missing parameter validation in `constructor`

Some parameters of constructors are not checked for invalid values.

#### <ins>Proof Of Concept</ins>

```solidity
36: address _owner

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol#L36

```solidity
54: address _owner

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L54



#### <ins>Recommended Mitigation Steps</ins>

Validate the parameters.



### <a href="#Summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> Missing Contract-existence Checks Before Low-level Calls

Low-level calls return success if there is no code present at the specified address. 

#### <ins>Proof Of Concept</ins>


```solidity
24: return target.call(callData);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/AdminProxy.sol#L24

```solidity
43: if (template.requiresInitData) (success, ) = clone.call(data);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneFactory.sol#L43






#### <ins>Recommended Mitigation Steps</ins>

In addition to the zero-address checks, add a check to verify that `<address>.code.length > 0`






### <a href="#Summary">[LOW&#x2011;6]</a><a name="LOW&#x2011;6"> Contracts are not using their OZ Upgradeable counterparts

The non-upgradeable standard version of OpenZeppelin’s library are inherited / used by the contracts.
It would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

#### <ins>Proof of Concept</ins>

```solidity
8: import { Math } from "openzeppelin-contracts/utils/math/Math.sol";

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L8

```solidity
6: import { Clones } from "openzeppelin-contracts/proxy/Clones.sol";

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneFactory.sol#L6

```solidity
10: import {IERC20Metadata} from "openzeppelin-contracts/token/ERC20/extensions/IERC20Metadata.sol";

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L10



#### <ins>Recommended Mitigation Steps</ins>

Where applicable, use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts`.
See https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/tree/master/contracts for list of available upgradeable contracts







### <a href="#Summary">[LOW&#x2011;7]</a><a name="LOW&#x2011;7"> Unbounded loop

New items are pushed into the following arrays:

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L263

MultiRewardStaking.accrueRewards() will iterate all the rewardTokens.
 
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L373

Currently, the array can grow indefinitely. E.g. there's no maximum limit and there's no functionality to remove array values.

If the array grows too large, calling MultiRewardStaking.accrueRewards() might run out of gas and revert. Calling these functions will result in a DOS condition.

#### <ins>Proof Of Concept</ins>


```solidity
263: rewardTokens.push(rewardToken);

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L263


In addition the following can push values in the array but there is no function to remove them:

```solidity
126: userEscrowIds[account].push(id);
127: userEscrowIdsByToken[account][token].push(id);

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L126-L127

```solidity
47: clones[templateCategory][templateId].push(clone);
48: allClones.push(clone);

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol#L47-L48

```solidity
56: templateCategories.push(templateCategory);

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol#L56

```solidity
78: templateIds[templateCategory].push(templateId);

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol#L78

```solidity
49: allVaults.push(_metadata.vault);
50: vaultsByAsset[IERC4626(_metadata.vault).asset()].push(_metadata.vault);

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol#L49-L50



#### <ins>Recommended Mitigation Steps</ins>
Add a functionality to delete array values or add a maximum size limit for arrays.



### <a href="#Summary">[LOW&#x2011;8]</a><a name="LOW&#x2011;8"> No storage gap for upgradeable contract might lead to storage slot collision

For upgradeable contracts, there must be storage gap to "allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments". Otherwise it may be very difficult to write new implementation code. Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract. This could have unintended and very serious consequences to the child contracts.

Refer to the bottom part of this article: https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable

#### <ins>Proof Of Concept</ins>

However, the contract doesn't contain a storage gap. The storage gap is essential for upgradeable contract because "It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments". See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

```solidity
ERC4626Upgradeable
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L26

```solidity
ERC20Upgradeable
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#0



#### <ins>Recommended Mitigation Steps</ins>
Recommend adding appropriate storage gap at the end of upgradeable contracts such as the below. Please reference OpenZeppelin upgradeable contract templates.
```solidity
    uint256[50] private __gap;
```



### <a href="#Summary">[LOW&#x2011;9]</a><a name="LOW&#x2011;9"> Upgrade OpenZeppelin Contract Dependency

An outdated OZ version is used (which has known vulnerabilities, see: https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories).

#### <ins>Proof Of Concept</ins>

Require dependencies to be at least version of 4.8.1 to include critical vulnerability patches.

```solidity
    "@openzeppelin/contracts": "4.8.0"
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/../package.json#L25



#### <ins>Recommended Mitigation Steps</ins>

Update OpenZeppelin Contracts Usage in package.json and require at least version of 4.8.1 via `>=4.8.1`





### <a href="#Summary">[LOW&#x2011;10]</a><a name="LOW&#x2011;10"> Missing `delegatecall` success check

The following function is missing a check on whether the return results of the `delegatecall` returns a success or failure

#### <ins>Proof Of Concept</ins>

```solidity
444: function harvest() public takeFees {
        if (
            address(strategy) != address(0) &&
            ((lastHarvest + harvestCooldown) < block.timestamp)
        ) {
            // solhint-disable
            address(strategy).delegatecall(
                abi.encodeWithSignature("harvest()")
            );
        }

        emit Harvested();
    }
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L438-L450

#### <ins>Recommended Mitigation Steps</ins>
Add a success check using: `(bool success, ) =` and adding an `if` condition for success or failure.


### <a href="#Summary">[LOW&#x2011;11]</a><a name="LOW&#x2011;11"> `SECONDS_PER_YEAR` value is incorrect

It is stated in https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L423-L428 that `Management fee is annualized per minute, based on 525,600 minutes per year`.
However, `SECONDS_PER_YEAR` is set to `365.25 days` which is `525960` minutes.

#### <ins>Proof Of Concept</ins>

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L35

#### <ins>Recommended Mitigation Steps</ins>

Set `SECONDS_PER_YEAR` to `525960` minutes or `365` days.



## Non Critical Issues

### <a href="#Summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). Consider adding a timelock to the following functions:

#### <ins>Proof Of Concept</ins>


```solidity
207: function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L207

```solidity
296: function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L296


```solidity
553: function setFeeRecipient(address _feeRecipient) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L553


```solidity
372: function changeVaultFees(address[] calldata vaults) external {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L372

```solidity
408: function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L408

```solidity
483: function changeStakingRewardsSpeeds(
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L483

```solidity
543: function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L543

```solidity
751: function setPerformanceFee(uint256 newFee) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L751

```solidity
764: function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L764





### <a href="#Summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Avoid Floating Pragmas: The Version Should Be Locked

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

#### <ins>Proof Of Concept</ins>

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [IEIP165.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IEIP165.sol#L2

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [IMultiRewardEscrow.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol#L3

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [IMultiRewardStaking.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L3

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [IAdapter.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IAdapter.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [IAdminProxy.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IAdminProxy.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [ICloneFactory.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ICloneFactory.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [ICloneRegistry.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ICloneRegistry.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [IDeploymentController.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IDeploymentController.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [IERC4626.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IERC4626.sol#L3

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [IPermissionRegistry.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IPermissionRegistry.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [IStrategy.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IStrategy.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [ITemplateRegistry.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ITemplateRegistry.sol#L3

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [IVault.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol#L3

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [IVaultController.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVaultController.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [IVaultRegistry.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVaultRegistry.sol#L3

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [IWithRewards.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IWithRewards.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [EIP165.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/EIP165.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [MultiRewardEscrow.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [MultiRewardStaking.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [AdminProxy.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/AdminProxy.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [CloneFactory.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneFactory.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [CloneRegistry.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [DeploymentController.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [PermissionRegistry.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [TemplateRegistry.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [Vault.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [VaultController.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L3

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [VaultRegistry.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [AdapterBase.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [OnlyStrategy.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/OnlyStrategy.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [WithRewards.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/WithRewards.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [BeefyAdapter.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [IBeefy.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/IBeefy.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [IYearn.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/IYearn.sol#L4

```solidity
Found usage of floating pragmas ^0.8.15 of Solidity in [YearnAdapter.sol]
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L4






### <a href="#Summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Variable Names That Consist Of All Capital Letters Should Be Reserved For Const/immutable Variables

If the variable needs to be different based on which class it comes from, a view/pure function should be used instead.

#### <ins>Proof Of Concept</ins>

```solidity
274: uint64 ONE = (10**IERC20Metadata(address(rewardToken)).decimals()).safeCastTo64();
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L274










### <a href="#Summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Critical Changes Should Use Two-step Procedure

The critical procedures should be two step process.

See similar findings in previous Code4rena contests for reference:
https://code4rena.com/reports/2022-06-illuminate/#2-critical-changes-should-use-two-step-procedure

#### <ins>Proof Of Concept</ins>

```solidity
207: function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L207

```solidity
296: function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L296


```solidity
553: function setFeeRecipient(address _feeRecipient) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L553


```solidity
372: function changeVaultFees(address[] calldata vaults) external {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L372

```solidity
408: function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L408

```solidity
483: function changeStakingRewardsSpeeds(
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L483

```solidity
543: function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L543

```solidity
751: function setPerformanceFee(uint256 newFee) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L751

```solidity
764: function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L764



#### <ins>Recommended Mitigation Steps</ins>

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.








### <a href="#Summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> block.timestamp is already used when emitting events, no need to input timestamp

#### <ins>Proof Of Concept</ins>

```solidity
536: emit NewFeesProposed(newFees, block.timestamp);

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L536

```solidity
585: emit NewAdapterProposed(newAdapter, block.timestamp);

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L585






### <a href="#Summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> Event emit should emit a parameter

Some emitted events do not have any emitted parameters. It is recommended to add some parameter such as state changes or value changes when events are emitted

#### <ins>Proof Of Concept</ins>


```solidity
449: emit Harvested()

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L449








### <a href="#Summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> Function writing that does not comply with the Solidity Style Guide

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:
- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private
- within a grouping, place the view and pure functions last

#### <ins>Proof Of Concept</ins>

All in-scope contracts




### <a href="#Summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> Use `delete` to Clear Variables

`delete a` assigns the initial value for the type to `a`. i.e. for integers it is equivalent to `a = 0`, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. Similarly, it can also be used to set an address to zero address. It has no effect on whole mappings though (as the keys of mappings may be arbitrary and are generally unknown). However, individual keys and what they map to can be deleted: If `a` is a mapping, then `delete a[x]` will delete the value stored at `x`.

The `delete` key better conveys the intention and is also more idiomatic. Consider replacing assignments of zero with `delete` statements.

#### <ins>Proof Of Concept</ins>

```solidity
186: accruedRewards[user][_rewardTokens[i]] = 0;

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L186

```solidity
577: underlyingBalance = 0;

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L577





### <a href="#Summary">[NC&#x2011;9]</a><a name="NC&#x2011;9"> Imports can be grouped together

Consider importing OZ first, then all interfaces, then all utils.

#### <ins>Proof Of Concept</ins>


```solidity
6: import { Owned } from "../utils/Owned.sol";
7: import { IOwned } from "../interfaces/IOwned.sol";
8: import { ICloneFactory } from "../interfaces/vault/ICloneFactory.sol";
9: import { ICloneRegistry } from "../interfaces/vault/ICloneRegistry.sol";
10: import { ITemplateRegistry, Template } from "../interfaces/vault/ITemplateRegistry.sol";

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol#L6-L10

```solidity
6: import { Owned } from "../utils/Owned.sol";
7: import { Permission } from "../interfaces/vault/IPermissionRegistry.sol";

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol#L6-L7

```solidity
6: import { Owned } from "../utils/Owned.sol";
7: import { Template } from "../interfaces/vault/ITemplateRegistry.sol";

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol#L6-L7

```solidity
6: import {SafeERC20Upgradeable as SafeERC20} from "openzeppelin-contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
7: import {ReentrancyGuardUpgradeable} from "openzeppelin-contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
8: import {PausableUpgradeable} from "openzeppelin-contracts-upgradeable/security/PausableUpgradeable.sol";
9: import {IERC4626, IERC20} from "../interfaces/vault/IERC4626.sol";
10: import {IERC20Metadata} from "openzeppelin-contracts/token/ERC20/extensions/IERC20Metadata.sol";
11: import {VaultFees} from "../interfaces/vault/IVault.sol";
12: import {MathUpgradeable as Math} from "openzeppelin-contracts-upgradeable/utils/math/MathUpgradeable.sol";
13: import {OwnedUpgradeable} from "../utils/OwnedUpgradeable.sol";
14: import {ERC20Upgradeable} from "openzeppelin-contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L6-L14

```solidity
5: import { SafeERC20Upgradeable as SafeERC20 } from "openzeppelin-contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
6: import { Owned } from "../utils/Owned.sol";
7: import { IVault, VaultInitParams, VaultFees } from "../interfaces/vault/IVault.sol";
8: import { IMultiRewardStaking } from "../interfaces/IMultiRewardStaking.sol";
9: import { IMultiRewardEscrow } from "../interfaces/IMultiRewardEscrow.sol";
10: import { IDeploymentController, ICloneRegistry } from "../interfaces/vault/IDeploymentController.sol";
11: import { ITemplateRegistry, Template } from "../interfaces/vault/ITemplateRegistry.sol";
12: import { IPermissionRegistry, Permission } from "../interfaces/vault/IPermissionRegistry.sol";
13: import { IVaultRegistry, VaultMetadata } from "../interfaces/vault/IVaultRegistry.sol";
14: import { IAdminProxy } from "../interfaces/vault/IAdminProxy.sol";
15: import { IERC4626, IERC20 } from "../interfaces/vault/IERC4626.sol";
16: import { IStrategy } from "../interfaces/vault/IStrategy.sol";
17: import { IAdapter } from "../interfaces/vault/IAdapter.sol";
18: import { IPausable } from "../interfaces/IPausable.sol";
19: import { DeploymentArgs } from "../interfaces/vault/IVaultController.sol";

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L5-L19

```solidity
6: import { IERC4626 } from "../interfaces/vault/IERC4626.sol";
7: import { Owned } from "../utils/Owned.sol";
8: import { VaultMetadata } from "../interfaces/vault/IVaultRegistry.sol";

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol#L6-L8

```solidity
6: import { EIP165 } from "../../../utils/EIP165.sol";
7: import { OnlyStrategy } from "./OnlyStrategy.sol";
8: import { IWithRewards } from "../../../interfaces/vault/IWithRewards.sol";
9: import { IAdapter } from "../../../interfaces/vault/IAdapter.sol";

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/WithRewards.sol#L6-L9

```solidity
6: import {AdapterBase, IERC20, IERC20Metadata, SafeERC20, ERC20, Math, IStrategy, IAdapter} from "../abstracts/AdapterBase.sol";
7: import {WithRewards, IWithRewards} from "../abstracts/WithRewards.sol";
8: import {IBeefyVault, IBeefyBooster, IBeefyBalanceCheck, IBeefyStrat} from "./IBeefy.sol";
9: import {IPermissionRegistry} from "../../../interfaces/vault/IPermissionRegistry.sol";

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L6-L9







### <a href="#Summary">[NC&#x2011;10]</a><a name="NC&#x2011;10"> NatSpec return parameters should be included in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html
#### <ins>Proof Of Concept</ins>


All in-scope contracts

i.e. missing NATSPEC

```solidity
  function _getClaimableAmount(Escrow memory escrow) internal view returns (uint256) {
    if (
      escrow.lastUpdateTime == 0 ||
      escrow.end == 0 ||
      escrow.balance == 0 ||
      block.timestamp.safeCastTo32() < escrow.start
    ) {
      return 0;
    }
    return
      Math.min(
        (escrow.balance * (block.timestamp - uint256(escrow.lastUpdateTime))) /
          (uint256(escrow.end) - uint256(escrow.lastUpdateTime)),
        escrow.balance
      );
  }
```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L170-L185

#### <ins>Recommended Mitigation Steps</ins>

Include return parameters in NatSpec comments

Recommendation Code Style: (from Uniswap3)

```solidity
    /// @notice Adds liquidity for the given recipient/tickLower/tickUpper position
    /// @dev The caller of this method receives a callback in the form of IUniswapV3MintCallback#uniswapV3MintCallback
    /// in which they must pay any token0 or token1 owed for the liquidity. The amount of token0/token1 due depends
    /// on tickLower, tickUpper, the amount of liquidity, and the current price.
    /// @param recipient The address for which the liquidity will be created
    /// @param tickLower The lower tick of the position in which to add liquidity
    /// @param tickUpper The upper tick of the position in which to add liquidity
    /// @param amount The amount of liquidity to mint
    /// @param data Any data that should be passed through to the callback
    /// @return amount0 The amount of token0 that was paid to mint the given amount of liquidity. Matches the value in the callback
    /// @return amount1 The amount of token1 that was paid to mint the given amount of liquidity. Matches the value in the callback
    function mint(
        address recipient,
        int24 tickLower,
        int24 tickUpper,
        uint128 amount,
        bytes calldata data
    ) external returns (uint256 amount0, uint256 amount1);
```



### <a href="#Summary">[NC&#x2011;11]</a><a name="NC&#x2011;11"> Insufficient coverage

The test coverage rate of the project is 94.52%. Testing all functions is best practice in terms of security criteria.
As stated in the docs: `What is the overall line coverage percentage provided by your tests?: 94.52%`

Due to its capacity, test coverage is expected to be 100%.



### <a href="#Summary">[NC&#x2011;12]</a><a name="NC&#x2011;12"> Lines are too long

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length
Reference: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

#### <ins>Proof Of Concept</ins>

```solidity
380: // If a deposit/withdraw operation gets called for another user we should accrue for both of them to avoid potential issues like in the Convex-Vulnerability
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L380





### <a href="#Summary">[NC&#x2011;13]</a><a name="NC&#x2011;13"> Missing event for critical parameter change

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

#### <ins>Proof Of Concept</ins>


```solidity
296: function changeRewardSpeed(IERC20 rewardToken, uint160 rewardsPerSecond) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L296

```solidity
335: function changeVaultAdapters(address[] calldata vaults) external {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L335

```solidity
372: function changeVaultFees(address[] calldata vaults) external {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L372

```solidity
408: function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L408

```solidity
483: function changeStakingRewardsSpeeds(
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L483

```solidity
543: function setEscrowTokenFees(IERC20[] calldata tokens, uint256[] calldata fees) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L543

```solidity
764: function setAdapterPerformanceFees(address[] calldata adapters) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L764

```solidity
804: function setAdapterHarvestCooldowns(address[] calldata adapters) external onlyOwner {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L804





### <a href="#Summary">[NC&#x2011;14]</a><a name="NC&#x2011;14"> Implementation contract may not be initialized

OpenZeppelin recommends that the initializer modifier be applied to constructors. 
Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.
https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5

#### <ins>Proof Of Concept</ins>


```solidity
30: constructor(address _owner, address _feeRecipient) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L30

```solidity
16: constructor(address _owner) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/AdminProxy.sol#L16

```solidity
23: constructor(address _owner) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneFactory.sol#L23

```solidity
22: constructor(address _owner) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol#L22

```solidity
35: constructor(
    address _owner,
    ICloneFactory _cloneFactory,
    ICloneRegistry _cloneRegistry,
    ITemplateRegistry _templateRegistry
  ) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol#L35

```solidity
20: constructor(address _owner) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol#L20

```solidity
24: constructor(address _owner) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol#L24

```solidity
53: constructor(
    address _owner,
    IAdminProxy _adminProxy,
    IDeploymentController _deploymentController,
    IVaultRegistry _vaultRegistry,
    IPermissionRegistry _permissionRegistry,
    IMultiRewardEscrow _escrow
  ) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L53

```solidity
21: constructor(address _owner) Owned(_owner)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol#L21





### <a href="#Summary">[NC&#x2011;15]</a><a name="NC&#x2011;15"> NatSpec comments should be increased in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

#### <ins>Proof Of Concept</ins>


All in-scope contracts

i.e. missing NATSPEC

```solidity
  function _getClaimableAmount(Escrow memory escrow) internal view returns (uint256) {
    if (
      escrow.lastUpdateTime == 0 ||
      escrow.end == 0 ||
      escrow.balance == 0 ||
      block.timestamp.safeCastTo32() < escrow.start
    ) {
      return 0;
    }
    return
      Math.min(
        (escrow.balance * (block.timestamp - uint256(escrow.lastUpdateTime))) /
          (uint256(escrow.end) - uint256(escrow.lastUpdateTime)),
        escrow.balance
      );
  }
```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L170-L185


#### <ins>Recommended Mitigation Steps</ins>

NatSpec comments should be increased in contracts



### <a href="#Summary">[NC&#x2011;16]</a><a name="NC&#x2011;16"> Use a more recent version of Solidity

<a href="https://blog.soliditylang.org/2022/08/08/solidity-0.8.16-release-announcement/">0.8.16</a>:

Code Generation: Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.

<a href="https://blog.soliditylang.org/2022/09/08/solidity-0.8.17-release-announcement/">0.8.17</a>:

Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.

#### <ins>Proof Of Concept</ins>


```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IEIP165.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardEscrow.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/IMultiRewardStaking.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IAdapter.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IAdminProxy.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ICloneFactory.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ICloneRegistry.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IDeploymentController.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IERC4626.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IPermissionRegistry.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IStrategy.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/ITemplateRegistry.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVault.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVaultController.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IVaultRegistry.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/interfaces/vault/IWithRewards.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/EIP165.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/AdminProxy.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneFactory.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/CloneRegistry.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/DeploymentController.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/PermissionRegistry.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/TemplateRegistry.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultController.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/VaultRegistry.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/OnlyStrategy.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/WithRewards.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/BeefyAdapter.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/beefy/IBeefy.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/IYearn.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/yearn/YearnAdapter.sol#L2



#### <ins>Recommended Mitigation Steps</ins>

Consider updating to a more recent solidity version.



### <a href="#Summary">[NC&#x2011;17]</a><a name="NC&#x2011;17"> Open TODOs

An open TODO is present. It is recommended to avoid open TODOs as they may indicate programming errors that still need to be fixed.

#### <ins>Proof Of Concept</ins>


```solidity
516: // TODO use deterministic fee recipient proxy
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L516













### <a href="#Summary">[NC&#x2011;18]</a><a name="NC&#x2011;18"> Use `bytes.concat()`

Solidity version 0.8.4 introduces `bytes.concat()` (vs `abi.encodePacked(<bytes>,<bytes>)`)

#### <ins>Proof Of Concept</ins>


```solidity
104: bytes32 id = keccak256(abi.encodePacked(token, account, amount, nonce)

```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardEscrow.sol#L104


#### <ins>Recommended Mitigation Steps</ins>

Use `bytes.concat()` and upgrade to at least Solidity version 0.8.4 if required. 



### <a href="#Summary">[NC&#x2011;19]</a><a name="NC&#x2011;19"> Use of Block.Timestamp

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.
References: SWC ID: 116

#### <ins>Proof Of Concept</ins>


```solidity
356: if (rewardsEndTimestamp > block.timestamp)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L356

```solidity
392: if (rewards.rewardsEndTimestamp > block.timestamp) {
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L392

```solidity
454: if (deadline < block.timestamp) revert PermitDeadlineExpired(deadline);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/utils/MultiRewardStaking.sol#L454

```solidity
488: if (managementFee > 0) feesUpdatedAt = block.timestamp;
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L488

```solidity
541: if (block.timestamp < proposedFeeTime + quitPeriod)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L541

```solidity
595: if (block.timestamp < proposedAdapterTime + quitPeriod)
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L595

```solidity
673: if (deadline < block.timestamp) revert PermitDeadlineExpired(deadline);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/Vault.sol#L673

```solidity
641: if (deadline < block.timestamp) revert PermitDeadlineExpired(deadline);
```

https://github.com/code-423n4/2023-01-popcorn/tree/main/src/vault/adapter/abstracts/AdapterBase.sol#L641



#### <ins>Recommended Mitigation Steps</ins>
Block timestamps should not be used for entropy or generating random numbers—i.e., they should not be the deciding factor (either directly or through some derivation) for winning a game or changing an important state.

Time-sensitive logic is sometimes required; e.g., for unlocking contracts (time-locking), completing an ICO after a few weeks, or enforcing expiry dates. It is sometimes recommended to use block.number and an average block time to estimate times; with a 10 second block time, 1 week equates to approximately, 60480 blocks. Thus, specifying a block number at which to change a contract state can be more secure, as miners are unable to easily manipulate the block number.



