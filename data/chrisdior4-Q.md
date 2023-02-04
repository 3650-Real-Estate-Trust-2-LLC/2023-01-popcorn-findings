# QA report

## Low Risk
| L-N    |Issue|Instances|
|:------:|:----|:-------:|
| [L&#x2011;01] | Signature malleability for `ecrecover` | 2 |
| [L&#x2011;02] | Decimals() not part of ERC20 standard | 2 |
| [L&#x2011;03] | Missing zero address check | 1 |
| [L&#x2011;04] | Insufficient input validation | 1 |

Total: 6 instances over 4 issues

## Non-critical

| N-N    |Issue|Instances|
|:------:|:----|:-------:|
| [N&#x2011;01] | Missing NatSpec documentation | 2 |
| [N&#x2011;02] | Do not use floating pragma, use a concrete version instead | 17 |
| [N&#x2011;03] | Unused custom error in `AdapterBase.sol` | 1 |
| [N&#x2011;04] | Typos in the comments | 3 |
| [N&#x2011;05] | Components in the contract are not neatly organised | 1 |
| [N&#x2011;06] | Open TODO in the code | 1 |

Total: 25 instances over 6 issues

## Low Risk

### \[L-01\]  Signature malleability for `ecrecover` 

Files: `Vault.sol` and `AdapterBase.sol`

The implementation of a cryptographic signature system in Ethereum contracts often assumes that the signature is unique, but signatures can be altered without the possession of the private key and still be valid. The EVM specification defines several so-called ‘precompiled’ contracts one of them being ecrecover which executes the elliptic curve public key recovery. A malicious user can slightly modify the three values v, r and s to create other valid signatures. A system that performs signature verification on contract level might be susceptible to attacks if the signature is part of the signed message hash. Valid signatures could be created by a malicious user to replay previously signed messages.

```solidity
address recoveredAddress = ecrecover(
```


```solidity
address recoveredAddress = ecrecover(
```


Lines of code: 

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L678

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L646


Use OpenZeppelin ECDSA library


### \[L-02\] Decimals() not part of ERC20 standard

File: `AdapterBase.sol`

The ` __ERC4626_init`  method is making a check if the smart contract of the underlying assset has decimals() implemented and if not - handles it.  But here:

```solidity
_decimals = IERC20Metadata(asset).decimals();
```
the code written in this way, assume  that the underlying asset will have  decimals() implemented. However decimals() is not part of the official ERC20 standard and might fall for tokens that do not implement it. While in practice it is very unlikely, as usually most of the tokens implement it, this should still be considered as a potential issue.

The same problem occurs also in `Vault.sol`

```solidity
  _decimals = IERC20Metadata(address(asset_)).decimals();
```

Lines of code for the two files respectively:

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L77

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L82


Consider using a helper function to safely retrieve token decimals.


### \[L-03\] Missing zero address check 

File: `AdapterBase.sol`

In `_withdraw()` assets are sent to `receiver` but his address is not checked. This way there is a risk for the funds to stuck forever in 0 address.

```solidity
IERC20(asset()).safeTransfer(receiver, assets);
```
Lines of code:

- https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L230

Add zero address check for `receiver` parameter in  `withdraw()`.

### \[L-04\] Insufficient input validation

In MultiRewardEscrow::lock we have parameter `duration` which only requirement is not to be equal to zero. Accidentally user can input 1 as an argument and if the start time is now and the offset is set to 0 (because we do not have ZeroAmount check for this param), the end time will be calculated by adding 1 to it, meaning only 1 second after the start, the funds will be available for transfer to `account`. 

```solidity
function lock(
    IERC20 token,
    address account,
    uint256 amount,
    uint32 duration,
    uint32 offset
  ) external {
    if (token == IERC20(address(0))) revert ZeroAddress();
    if (account == address(0)) revert ZeroAddress();
    if (amount == 0) revert ZeroAmount();
    if (duration == 0) revert ZeroAmount();
```

```solidity
uint32 start = block.timestamp.safeCastTo32() + offset;

    escrows[id] = Escrow({
      token: token,
      start: start,
      end: start + duration,
      lastUpdateTime: start,
      initialBalance: amount,
      balance: amount,
      account: account
    });
```


If this is not a problem for the protocol, remove the duration should be different from 0 check, because it will make no big difference, if the funds will be unlocked `now` or `now` + 1 second.

## Non-critical

### \[NC-01\] Missing NatSpec documentation

File: `AdapterBase.sol`

There are couple of places that @param field in the NatSpec is missing. Here is an example:

```solidity
/**
* @notice Allows the strategy to withdraw assets from the underlying protocol without burning adapter shares.
* @dev This can be used e.g. for a leverage strategy to reduce leverage without the need for the strategy to hold any adapter shares.
*/
function strategyWithdraw(uint256 amount, uint256 shares)
        public
        onlyStrategy
    {
        _protocolWithdraw(amount, shares);
    }
```

Consider adding full NatSpec documentation in all the contracts.


### \[NC-02\] Do not use floating pragma, use a concrete version instead

Currently the smart contracts are using a floating pragma, change it to concrete 0.8.18 version.

### \[NC-03\] Unused custom error in `AdapterBase.sol`

```solidity
error ZeroAmount();
```

Consider using it or removing it from the contract.


### \[NC-04\] Typos in the comments

File: `VaultController.sol`

`ownes` -> `owns`
`informations` -> `information`

File: `YearnAdapter.sol`

`profts` -> `profits`

###  \[NC-05\] Components in the contract are not neatly organised

File: `AdapterBase.sol`

Its best practice to group all events at one place, as well as custom errors and put them in the interface of the implementation contract, or at least put them right after the contract declaration, before the functions. Here the declarations are not well structured.

`BeefyAdapte.sol` is a perfect example of how the contracts should be organised

###  \[NC-06\]  Open TODO in the code

There is an open TODO in `AdapterBase.sol`  this implies changes that might not be audited. Resolve it or remove it

```solidity
 // TODO use deterministic fee recipient proxy
 ```
 
 Lines of code: 
 
 - https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L516
