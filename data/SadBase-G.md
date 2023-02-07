## Summary

### Gas Optimizations

|                               | Issue                                                                                        | Contexts | Estimated Gas Saved |
| ----------------------------- |:-------------------------------------------------------------------------------------------- |:-------- |:-------------------:|
| G-01 | Changing variables to `constant` from `immutable` for variables initialized saves gas        | 5        |          -          |
| G-02 | State variables only set in the `constructor` should be declared immutable                   | 8        |        8000         |
| G-03 | Remove Unused Local Variables                                                                | 1        |          -          |
| G-04 | Remove Unused Arguments                                                                      | 1        |          -          |
| G-05 | Unnecessary Typecasting                                                                      | 1        |          -          |
| G-06 | State variables should be cached in stack variables rather than re-reading them from storage | 1        |          -          |
| G-07 | Gas Usage can be reduced with View Functions for Non-State Modifying Operations              | 2        |          -          |
| G-08 | Variable Does Not Need to Be Cached if Only Used Once                                        | 1        |          -          |
| G-09 | Functions guaranteed to revert when called by normal users can be marked `payable`           | 1        |          -          |

21 contexts over 9 issues

## Gas Optimizations

### [G-01] Changing variables to `constant` from `immutable` for variables initialized saves gas

#### Proof Of Concept

_There are 5 instances of this issue:_

```solidity
File: src/vault/VaultController.sol

36:  bytes32 public immutable VAULT = "Vault";
37:  bytes32 public immutable ADAPTER = "Adapter";
38:  bytes32 public immutable STRATEGY = "Strategy";
39:  bytes32 public immutable STAKING = "Staking";
40:  bytes4 internal immutable DEPLOY_SIG = bytes4(keccak256("deploy(bytes32,bytes32,bytes)"));
```

Link: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L36-L40

These variables are initialized and declared at compile time. It is better and more cost-saving to declare them as `constant`. Declaring it `private` after changing the variable type to `constant` will save more gas.

Example:

```solidity
36:  bytes32 private constant VAULT = "Vault";
37:  bytes32 private constant ADAPTER = "Adapter";
38:  bytes32 private constant STRATEGY = "Strategy";
39:  bytes32 private constant STAKING = "Staking";
40:  bytes4 internal constant DEPLOY_SIG = bytes4(keccak256("deploy(bytes32,bytes32,bytes)"));
```

### [G-02] State variables only set in the `constructor` should be declared immutable

The constructor utilizes a Gsset instruction, which is replaced with a more efficient PUSH32 instruction, resulting in a reduction of gas usage from 20,000 to 3 per operation. Additionally, instances of the Gwarmacces instruction (100 gas per execution) have been replaced with PUSH32, leading to a significant overall decrease in gas consumption.

#### Proof Of Concept

_There are 8 instances of this issue:_

```solidity
File: src/utils/MultiRewardEscrow.sol

191:  address public feeRecipient;
```

Link: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L191

```solidity
File: src/vault/DeploymentController.sol

23:  ICloneFactory public cloneFactory;
24:  ICloneRegistry public cloneRegistry;
25:  ITemplateRegistry public templateRegistry;
```

Link: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/DeploymentController.sol#L23-L25

```solidity
File: src/vault/VaultController.sol

387:  IVaultRegistry public vaultRegistry;

535:  IMultiRewardEscrow public escrow;

717:  IAdminProxy public immutable adminProxy;

822:  IPermissionRegistry public permissionRegistry;
```

Link: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L387

Link: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L535

Link: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L717

Link: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L822

### [G-03] Remove Unused Local Variables

#### Proof Of Concept

_There are 1 instances of this issue:_

```solidity
File: src/vault/Vault.sol

448:        uint256 highWaterMark_ = highWaterMark;
```

Link: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L448

### [G-04] Remove Unused Arguments

#### Proof Of Concept

_There are 1 instance of this issue:_

```solidity
File: src/vault/VaultController.sol

390:  function _registerVault(address vault, VaultMetadata memory metadata) internal {
```

Link: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L390

In this case the address `vault` is not used in the function.

### [G-05] Unnecessary Typecasting

#### Proof Of Concept

There is no need to typecast when the variable is already of type `address`.

_There are 1 instance of this issue:_

```solidity
File: src/vault/VaultController.sol

589:        address(_deploymentController),
```

Link: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L589

### [G-06] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

#### Proof Of Concept

_There are 1 instance of this issue:_

```solidity
File: src/vault/Vault.sol

448:        uint256 highWaterMark_ = highWaterMark;
```

Link: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L448

Use the local variable that has been cached inside the function.

##### Mitigation

```solidity
function accruedPerformanceFee() public view returns (uint256) {
	uint256 highWaterMark_ = highWaterMark;
	uint256 shareValue = convertToAssets(1e18);
	uint256 performanceFee = fees.performance;

	return
		performanceFee > 0 && shareValue > highWaterMark_
			? performanceFee.mulDiv(
				(shareValue - highWaterMark_) * totalSupply(),
				1e36,
				Math.Rounding.Down
			)
			: 0;
    }
```

### [G-07] Gas Usage can be reduced with View Functions for Non-State Modifying Operations

#### Proof Of Concept

_There are 2 instances of this issue:_

```solidity
File: src/vault/VaultController.sol

function _encodeAdapterData(DeploymentArgs memory adapterData, bytes memory baseAdapterData)
internal
returns (bytes memory)
{
	return
	  abi.encodeWithSelector(
		IAdapter.initialize.selector,
		baseAdapterData,
		templateRegistry.getTemplate(ADAPTER, adapterData.id).registry,
		adapterData.data
	  );
}
```

Link: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L242-L253

```solidity
File: src/vault/VaultController.sol

function _verifyCreatorOrOwner(address vault) internal returns (VaultMetadata memory metadata) {
metadata = vaultRegistry.getVault(vault);
if (msg.sender != metadata.creator || msg.sender != owner) revert NotSubmitterNorOwner(msg.sender);
}
```

Link: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L667-L670

##### Mitigation

```diff
function _encodeAdapterData(DeploymentArgs memory adapterData, bytes memory baseAdapterData)
internal
+ view
returns (bytes memory)
{
	return
	  abi.encodeWithSelector(
		IAdapter.initialize.selector,
		baseAdapterData,
		templateRegistry.getTemplate(ADAPTER, adapterData.id).registry,
		adapterData.data
	  );
}
```

```diff
+ function _verifyCreatorOrOwner(address vault) internal view returns (VaultMetadata memory metadata) {
metadata = vaultRegistry.getVault(vault);
if (msg.sender != metadata.creator || msg.sender != owner) revert NotSubmitterNorOwner(msg.sender);
}
```

### [G-08] Variable Does Not Need to Be Cached if Only Used Once

Removing redundant caching of variables can reduce gas usage. Since the state variable is only used once, gas can be saved by removing `MLOAD` and `MSTORE`.

#### Proof Of Concept

Removing this statement and simply accessing the state variable can save gas due to the fact that it is only accessed once. This gas optimization is linked to [G-05] where it is unncessary to typecast again when the variable is already of type `address`.

_There are 1 instance of this issue:_

```solidity
File: src/vault/VaultController.sol

586:    address _deploymentController = address(deploymentController);
```

Link: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L586

##### Mitigation

Remove declaring of local variable and directly call `address(deploymentController)` in line 589.

### [G-09] Using `private` rather than `public` for constants, saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table.

#### Proof Of Concept

_There are 1 instance of this issue:_

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

31:     uint256 public constant BPS_DENOMINATOR = 10_000;
```

Link: https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L31
