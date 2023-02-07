## Summary

### Low Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | Unchecked return value of low level `call()`/`delegatecall()` | 2 | 
| [L&#x2011;02] | Upgradeable contract not initialized | 2 | 
| [L&#x2011;03] | Loss of precision | 2 | 
| [L&#x2011;04] | Signatures vulnerable to malleability attacks | 3 | 
| [L&#x2011;05] | Owner can renounce while system is paused | 2 | 
| [L&#x2011;06] | `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()` | 1 | 
| [L&#x2011;07] | Open TODOs | 1 | 

Total: 13 instances over 7 issues


### Non-critical Issues
| |Issue|Instances|
|-|:-|:-:|
| [N&#x2011;01] | Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions | 2 | 
| [N&#x2011;02] | Missing `initializer` modifier on constructor | 1 | 
| [N&#x2011;03] | Unused file | 1 | 
| [N&#x2011;04] | `constant`s should be defined rather than using magic numbers | 36 | 
| [N&#x2011;05] | Events that mark critical parameter changes should contain both the old and the new value | 2 | 
| [N&#x2011;06] | Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`) | 1 | 
| [N&#x2011;07] | Lines are too long | 3 | 
| [N&#x2011;08] | Variable names that consist of all capital letters should be reserved for `constant`/`immutable` variables | 9 | 
| [N&#x2011;09] | Non-library/interface files should use fixed compiler versions, not floating ones | 16 | 
| [N&#x2011;10] | Typos | 8 | 
| [N&#x2011;11] | File is missing NatSpec | 14 | 
| [N&#x2011;12] | NatSpec is incomplete | 18 | 
| [N&#x2011;13] | Not using the named return variables anywhere in the function is confusing | 3 | 
| [N&#x2011;14] | Consider using `delete` rather than assigning zero to clear values | 1 | 
| [N&#x2011;15] | Contracts should have full test coverage | 1 | 
| [N&#x2011;16] | Large or complicated code bases should implement fuzzing tests | 1 | 
| [N&#x2011;17] | Function ordering does not follow the Solidity style guide | 33 | 
| [N&#x2011;18] | Contract does not follow the Solidity style guide's suggested layout ordering | 35 | 
| [N&#x2011;19] | Interfaces should be indicated with an `I` prefix in the contract name | 1 | 

Total: 186 instances over 19 issues





## Low Risk Issues

### [L&#x2011;01]  Unchecked return value of low level `call()`/`delegatecall()`

*There are 2 instances of this issue:*

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

444               address(strategy).delegatecall(
445                   abi.encodeWithSignature("harvest()")
446:              );

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L444-L446

```solidity
File: src/vault/AdminProxy.sol

24:       return target.call(callData);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/AdminProxy.sol#L24

### [L&#x2011;02]  Upgradeable contract not initialized
Upgradeable contracts are initialized via an initializer function rather than by a constructor. Leaving such a contract uninitialized may lead to it being taken over by a malicious user

*There are 2 instances of this issue:*

```solidity
File: src/vault/Vault.sol

/// @audit missing __ReentrancyGuard_init()
/// @audit missing __Pausable_init()
26:   contract Vault is

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L26

### [L&#x2011;03]  Loss of precision
Division by large numbers may result in the result being zero, due to solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator

*There are 2 instances of this issue:*

```solidity
File: src/utils/MultiRewardStaking.sol

359:      return (block.timestamp + (amount / uint256(rewardsPerSecond))).safeCastTo32();

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L359

```solidity
File: src/vault/Vault.sol

433                   ? managementFee.mulDiv(
434                       totalAssets() * (block.timestamp - feesUpdatedAt),
435                       SECONDS_PER_YEAR,
436                       Math.Rounding.Down
437:                  ) / 1e18

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L433-L437

### [L&#x2011;04]  Signatures vulnerable to malleability attacks
`ecrecover()` accepts as valid, two versions of signatures, meaning an attacker can use the same signature twice. Consider adding checks for signature malleability, or using OpenZeppelin's `ECDSA` library to perform the extra checks necessary in order to prevent this attack.

*There are 3 instances of this issue:*

```solidity
File: src/utils/MultiRewardStaking.sol

459         address recoveredAddress = ecrecover(
460           keccak256(
461             abi.encodePacked(
462               "\x19\x01",
463               DOMAIN_SEPARATOR(),
464               keccak256(
465                 abi.encode(
466                   keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),
467                   owner,
468                   spender,
469                   value,
470                   nonces[owner]++,
471                   deadline
472                 )
473               )
474             )
475           ),
476           v,
477           r,
478           s
479:        );

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L459-L479

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

646               address recoveredAddress = ecrecover(
647                   keccak256(
648                       abi.encodePacked(
649                           "\x19\x01",
650                           DOMAIN_SEPARATOR(),
651                           keccak256(
652                               abi.encode(
653                                   keccak256(
654                                       "Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"
655                                   ),
656                                   owner,
657                                   spender,
658                                   value,
659                                   nonces[owner]++,
660                                   deadline
661                               )
662                           )
663                       )
664                   ),
665                   v,
666                   r,
667                   s
668:              );

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L646-L668

```solidity
File: src/vault/Vault.sol

678               address recoveredAddress = ecrecover(
679                   keccak256(
680                       abi.encodePacked(
681                           "\x19\x01",
682                           DOMAIN_SEPARATOR(),
683                           keccak256(
684                               abi.encode(
685                                   keccak256(
686                                       "Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"
687                                   ),
688                                   owner,
689                                   spender,
690                                   value,
691                                   nonces[owner]++,
692                                   deadline
693                               )
694                           )
695                       )
696                   ),
697                   v,
698                   r,
699                   s
700:              );

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L678-L700

### [L&#x2011;05]  Owner can renounce while system is paused
The contract owner or single user with a role is not prevented from renouncing the role/ownership while the contract is paused, which would cause any user assets stored in the protocol, to be locked indefinitely

*There are 2 instances of this issue:*

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

574       function pause() external onlyOwner {
575           _protocolWithdraw(totalAssets(), totalSupply());
576           // Update the underlying balance to prevent inflation attacks
577           underlyingBalance = 0;
578           _pause();
579:      }

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L574-L579

```solidity
File: src/vault/Vault.sol

643       function pause() external onlyOwner {
644           _pause();
645:      }

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L643-L645

### [L&#x2011;06]  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). "Unless there is a compelling reason, `abi.encode` should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).

If all arguments are strings and or bytes, `bytes.concat()` should be used instead

*There is 1 instance of this issue:*

```solidity
File: src/vault/Vault.sol

93            contractName = keccak256(
94                abi.encodePacked("Popcorn", name(), block.timestamp, "Vault")
95:           );

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L93-L95

### [L&#x2011;07]  Open TODOs
Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment

*There is 1 instance of this issue:*

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

516:      // TODO use deterministic fee recipient proxy

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L516

## Non-critical Issues

### [N&#x2011;01]  Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions
See [this](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) link for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

*There are 2 instances of this issue:*

```solidity
File: src/utils/MultiRewardStaking.sol

26:   contract MultiRewardStaking is ERC4626Upgradeable, OwnedUpgradeable {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L26

```solidity
File: src/vault/Vault.sol

26    contract Vault is
27        ERC20Upgradeable,
28        ReentrancyGuardUpgradeable,
29        PausableUpgradeable,
30:       OwnedUpgradeable

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L26-L30

### [N&#x2011;02]  Missing `initializer` modifier on constructor
OpenZeppelin [recommends](https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5) that the `initializer` modifier be applied to constructors in order to avoid potential griefs, [social engineering](https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/4), or exploits. Ensure that the modifier is applied to the implementation contract. If the default constructor is currently being used, it should be changed to be an explicit one with the modifier applied.

*There is 1 instance of this issue:*

```solidity
File: src/vault/Vault.sol

28:       ReentrancyGuardUpgradeable,

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L28

### [N&#x2011;03]  Unused file
The file is never imported by any other source file. If the file is needed for tests, it should be moved to a test directory

*There is 1 instance of this issue:*

```solidity
File: src/interfaces/IEIP165.sol

1:    // SPDX-License-Identifier: AGPL-3.0-only

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/IEIP165.sol#L1

### [N&#x2011;04]  `constant`s should be defined rather than using magic numbers
Even [assembly](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

*There are 36 instances of this issue:*

```solidity
File: src/interfaces/vault/IStrategy.sol

/// @audit 8
9:      function verifyAdapterSelectorCompatibility(bytes4[8] memory sigs) external;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IStrategy.sol#L9

```solidity
File: src/interfaces/vault/ITemplateRegistry.sol

/// @audit 8
20:     bytes4[8] requiredSigs;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/ITemplateRegistry.sol#L20

```solidity
File: src/interfaces/vault/IVaultRegistry.sol

/// @audit 8
17:     address[8] swapTokenAddresses;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IVaultRegistry.sol#L17

```solidity
File: src/utils/MultiRewardEscrow.sol

/// @audit 1e18
108:        uint256 fee = Math.mulDiv(amount, feePerc, 1e18);

/// @audit 1e17
211:        if (tokenFees[i] >= 1e17) revert DontGetGreedy(tokenFees[i]);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L108

```solidity
File: src/utils/MultiRewardStaking.sol

/// @audit 1e18
197:      uint256 escrowed = rewardAmount.mulDiv(uint256(escrowInfo.escrowPercentage), 1e18, Math.Rounding.Down);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L197

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

/// @audit 8
64:               bytes4[8] memory _requiredSigs,

/// @audit 8
68:                   (address, address, address, uint256, bytes4[8], bytes)

/// @audit 1e18
85:           highWaterMark = 1e18;

/// @audit 8
479:      function _verifyAndSetupStrategy(bytes4[8] memory requiredSigs) internal {

/// @audit 1e18
531:          uint256 shareValue = convertToAssets(1e18);

/// @audit 1e36
538:                      1e36,

/// @audit 2e17
551:          if (newFee > 2e17) revert InvalidPerformanceFee(newFee);

/// @audit 1e18
565:          uint256 shareValue = convertToAssets(1e18);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L64

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

/// @audit 8
41:               (address, address, address, uint256, bytes4[8], bytes)

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L41

```solidity
File: src/vault/VaultController.sol

/// @audit 8
210:      bytes4[8] memory requiredSigs;

/// @audit 2e17
753:      if (newFee > 2e17) revert InvalidPerformanceFee(newFee);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L210

```solidity
File: src/vault/Vault.sol

/// @audit 1e18
144:              assets.mulDiv(uint256(fees.deposit), 1e18, Math.Rounding.Down)

/// @audit 1e18
183:              1e18 - depositFee,

/// @audit 1e18
224:              1e18 - withdrawalFee,

/// @audit 1e18
265:              1e18,

/// @audit 1e18
330:                  assets.mulDiv(uint256(fees.deposit), 1e18, Math.Rounding.Down)

/// @audit 1e18
345:              1e18 - depositFee,

/// @audit 1e18
367:              1e18 - withdrawalFee,

/// @audit 1e18
389:              1e18,

/// @audit 1e18
437:                  ) / 1e18

/// @audit 1e18
449:          uint256 shareValue = convertToAssets(1e18);

/// @audit 1e36
456:                      1e36,

/// @audit 1e18
484:          uint256 shareValue = convertToAssets(1e18);

/// @audit 1e18
498:          highWaterMark = convertToAssets(1e18);

/// @audit 1e18
527:              newFees.deposit >= 1e18 ||

/// @audit 1e18
528:              newFees.withdrawal >= 1e18 ||

/// @audit 1e18
529:              newFees.management >= 1e18 ||

/// @audit 1e18
530:              newFees.performance >= 1e18

/// @audit 3
619:      uint256 public quitPeriod = 3 days;

/// @audit 7
630:          if (_quitPeriod < 1 days || _quitPeriod > 7 days)

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L144

### [N&#x2011;05]  Events that mark critical parameter changes should contain both the old and the new value
This should especially be done if the new value is not required to be different from the old value

*There are 2 instances of this issue:*

```solidity
File: src/utils/MultiRewardEscrow.sol

/// @audit setFees()
214:        emit FeeSet(tokens[i], tokenFees[i]);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L214

```solidity
File: src/vault/Vault.sol

/// @audit setQuitPeriod()
635:          emit QuitPeriodSet(quitPeriod);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L635

### [N&#x2011;06]  Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`)
While the compiler knows to optimize away the exponentiation, it's still better coding practice to use idioms that do not require compiler optimization, if they exist

*There is 1 instance of this issue:*

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

25:       uint256 constant DEGRADATION_COEFFICIENT = 10**18;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L25

### [N&#x2011;07]  Lines are too long
Usually lines in source code are limited to [80](https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width) characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over [164](https://github.com/aizatto/character-length) characters, the lines below should be split when they reach that length

*There are 3 instances of this issue:*

```solidity
File: src/utils/MultiRewardEscrow.sol

49:      * @dev there is no check to ensure that all escrows are owned by the same account. Make sure to account for this either by only sending ids for a specific account or by filtering the Escrows by account later on.

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L49

```solidity
File: src/utils/MultiRewardStaking.sol

7:    import { ERC4626Upgradeable, ERC20Upgradeable, IERC20Upgradeable as IERC20, IERC20MetadataUpgradeable as IERC20Metadata } from "openzeppelin-contracts-upgradeable/token/ERC20/extensions/ERC4626Upgradeable.sol";

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L7

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

6:    import {ERC4626Upgradeable, IERC20Upgradeable as IERC20, IERC20MetadataUpgradeable as IERC20Metadata, ERC20Upgradeable as ERC20} from "openzeppelin-contracts-upgradeable/token/ERC20/extensions/ERC4626Upgradeable.sol";

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L6

### [N&#x2011;08]  Variable names that consist of all capital letters should be reserved for `constant`/`immutable` variables
If the variable needs to be different based on which class it comes from, a `view`/`pure` _function_ should be used instead (e.g. like [this](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/76eee35971c2541585e05cbf258510dda7b2fbc6/contracts/token/ERC20/extensions/draft-IERC20Permit.sol#L59)).

*There are 9 instances of this issue:*

```solidity
File: src/interfaces/IMultiRewardStaking.sol

16:     uint64 ONE;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/IMultiRewardStaking.sol#L16

```solidity
File: src/utils/MultiRewardStaking.sol

274:      uint64 ONE = (10**IERC20Metadata(address(rewardToken)).decimals()).safeCastTo64();

438:    uint256 internal INITIAL_CHAIN_ID;

439:    bytes32 internal INITIAL_DOMAIN_SEPARATOR;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L274

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

517:      address FEE_RECIPIENT = address(0x4444);

625:      uint256 internal INITIAL_CHAIN_ID;

626:      bytes32 internal INITIAL_DOMAIN_SEPARATOR;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L517

```solidity
File: src/vault/Vault.sol

657:      uint256 internal INITIAL_CHAIN_ID;

658:      bytes32 internal INITIAL_DOMAIN_SEPARATOR;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L657

### [N&#x2011;09]  Non-library/interface files should use fixed compiler versions, not floating ones

*There are 16 instances of this issue:*

```solidity
File: src/utils/EIP165.sol

4:    pragma solidity ^0.8.15;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/EIP165.sol#L4

```solidity
File: src/utils/MultiRewardEscrow.sol

4:    pragma solidity ^0.8.15;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L4

```solidity
File: src/utils/MultiRewardStaking.sol

4:    pragma solidity ^0.8.15;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L4

```solidity
File: src/vault/adapter/abstracts/OnlyStrategy.sol

4:    pragma solidity ^0.8.15;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/OnlyStrategy.sol#L4

```solidity
File: src/vault/adapter/abstracts/WithRewards.sol

4:    pragma solidity ^0.8.15;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/WithRewards.sol#L4

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

4:    pragma solidity ^0.8.15;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L4

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

4:    pragma solidity ^0.8.15;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L4

```solidity
File: src/vault/AdminProxy.sol

4:    pragma solidity ^0.8.15;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/AdminProxy.sol#L4

```solidity
File: src/vault/CloneFactory.sol

4:    pragma solidity ^0.8.15;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneFactory.sol#L4

```solidity
File: src/vault/CloneRegistry.sol

4:    pragma solidity ^0.8.15;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneRegistry.sol#L4

```solidity
File: src/vault/DeploymentController.sol

4:    pragma solidity ^0.8.15;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/DeploymentController.sol#L4

```solidity
File: src/vault/PermissionRegistry.sol

4:    pragma solidity ^0.8.15;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L4

```solidity
File: src/vault/TemplateRegistry.sol

4:    pragma solidity ^0.8.15;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L4

```solidity
File: src/vault/VaultController.sol

3:    pragma solidity ^0.8.15;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L3

```solidity
File: src/vault/VaultRegistry.sol

4:    pragma solidity ^0.8.15;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L4

```solidity
File: src/vault/Vault.sol

4:    pragma solidity ^0.8.15;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L4

### [N&#x2011;10]  Typos

*There are 8 instances of this issue:*

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

/// @audit overriden
24:    * All specific interactions for the underlying protocol need to be overriden in the actual implementation.

/// @audit aftwards
477:       * @dev It aftwards sets up anything required by the strategy to call `harvest()` like approvals etc.

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L24

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

/// @audit profts
100:      /// @notice The amount of assets that are free to be withdrawn from the yVault after locked profts.

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L100

```solidity
File: src/vault/AdminProxy.sol

/// @audit Ownes
11:    * @notice  Ownes contracts in the vault ecosystem to allow for easy ownership changes.

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/AdminProxy.sol#L11

```solidity
File: src/vault/VaultController.sol

/// @audit ownes
47:      * @param _adminProxy `AdminProxy` ownes contracts in the vault ecosystem.

/// @audit auxiliery
87:      * @dev This function is the one stop solution to create a new vault with all necessary admin functions or auxiliery contracts.

/// @audit DEPLYOMENT
816:                        DEPLYOMENT CONTROLLER LOGIC

/// @audit auxilary
829:     * @notice Sets a new `DeploymentController` and saves its auxilary contracts. Caller must be owner.

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L47

### [N&#x2011;11]  File is missing NatSpec

*There are 14 instances of this issue:*

```solidity
File: src/interfaces/IEIP165.sol


```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/IEIP165.sol

```solidity
File: src/interfaces/vault/IAdapter.sol


```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IAdapter.sol

```solidity
File: src/interfaces/vault/IAdminProxy.sol


```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IAdminProxy.sol

```solidity
File: src/interfaces/vault/ICloneFactory.sol


```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/ICloneFactory.sol

```solidity
File: src/interfaces/vault/ICloneRegistry.sol


```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/ICloneRegistry.sol

```solidity
File: src/interfaces/vault/IDeploymentController.sol


```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IDeploymentController.sol

```solidity
File: src/interfaces/vault/IERC4626.sol


```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IERC4626.sol

```solidity
File: src/interfaces/vault/IPermissionRegistry.sol


```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IPermissionRegistry.sol

```solidity
File: src/interfaces/vault/IStrategy.sol


```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IStrategy.sol

```solidity
File: src/interfaces/vault/IVaultController.sol


```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IVaultController.sol

```solidity
File: src/interfaces/vault/IWithRewards.sol


```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IWithRewards.sol

```solidity
File: src/utils/EIP165.sol


```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/EIP165.sol

```solidity
File: src/vault/adapter/beefy/IBeefy.sol


```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/IBeefy.sol

```solidity
File: src/vault/adapter/yearn/IYearn.sol


```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/IYearn.sol

### [N&#x2011;12]  NatSpec is incomplete

*There are 18 instances of this issue:*

```solidity
File: src/utils/MultiRewardEscrow.sol

/// @audit Missing: '@return'
49       * @dev there is no check to ensure that all escrows are owned by the same account. Make sure to account for this either by only sending ids for a specific account or by filtering the Escrows by account later on.
50       */
51:     function getEscrows(bytes32[] calldata escrowIds) external view returns (Escrow[] memory) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L49-L51

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

/// @audit Missing: '@return'
108        * @param receiver Receiver of the shares.
109        */
110       function deposit(uint256 assets, address receiver)
111           public
112           virtual
113           override
114:          returns (uint256)

/// @audit Missing: '@return'
127        * @param receiver Receiver of the shares.
128        */
129       function mint(uint256 shares, address receiver)
130           public
131           virtual
132           override
133:          returns (uint256)

/// @audit Missing: '@return'
171        * @param owner Owner of the shares.
172        */
173       function withdraw(
174           uint256 assets,
175           address receiver,
176           address owner
177:      ) public virtual override returns (uint256) {

/// @audit Missing: '@return'
191        * @param owner Owner of the shares.
192        */
193       function redeem(
194           uint256 shares,
195           address receiver,
196           address owner
197:      ) public virtual override returns (uint256) {

/// @audit Missing: '@param address'
399       /**
400        * @return Maximum amount of vault shares that may be minted to given address. Delegates to adapter.
401        * @dev Return 0 if paused since no further deposits are allowed.
402        * @dev Override this function if the underlying protocol has a unique deposit logic and/or deposit fees.
403        */
404       function maxDeposit(address)
405           public
406           view
407           virtual
408           override
409:          returns (uint256)

/// @audit Missing: '@param address'
414       /**
415        * @return Maximum amount of vault shares that may be minted to given address. Delegates to adapter.
416        * @dev Return 0 if paused since no further deposits are allowed.
417        * @dev Override this function if the underlying protocol has a unique deposit logic and/or deposit fees.
418        */
419:      function maxMint(address) public view virtual override returns (uint256) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L108-L114

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

/// @audit Missing: '@param bytes'
27        /**
28         * @notice Initialize a new Yearn Adapter.
29         * @param adapterInitData Encoded data for the base adapter initialization.
30         * @param externalRegistry Yearn registry address.
31         * @dev This function is called by the factory contract when deploying a new vault.
32         * @dev The yearn registry will be used given the `asset` from `adapterInitData` to find the latest yVault.
33         */
34        function initialize(
35            bytes memory adapterInitData,
36            address externalRegistry,
37            bytes memory
38:       ) external initializer {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L27-L38

```solidity
File: src/vault/CloneFactory.sol

/// @audit Missing: '@return'
37       * @param data The data to pass to the clone's initializer.
38       */
39:     function deploy(Template calldata template, bytes calldata data) external onlyOwner returns (address clone) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneFactory.sol#L37-L39

```solidity
File: src/vault/DeploymentController.sol

/// @audit Missing: '@return'
97       * @dev Registers the clone in `CloneRegistry`.
98       */
99      function deploy(
100       bytes32 templateCategory,
101       bytes32 templateId,
102       bytes calldata data
103:    ) external onlyOwner returns (address clone) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/DeploymentController.sol#L97-L103

```solidity
File: src/vault/VaultController.sol

/// @audit Missing: '@return'
87       * @dev This function is the one stop solution to create a new vault with all necessary admin functions or auxiliery contracts.
88       */
89      function deployVault(
90        VaultInitParams memory vaultData,
91        DeploymentArgs memory adapterData,
92        DeploymentArgs memory strategyData,
93        address staking,
94        bytes memory rewardsData,
95        VaultMetadata memory metadata,
96        uint256 initialDeposit
97:     ) external canCreate returns (address vault) {

/// @audit Missing: '@param initialDeposit'
/// @audit Missing: '@return'
180     /**
181      * @notice Deploy a new Adapter with our without a strategy. Caller must be owner.
182      * @param asset Asset which will be used by the adapter.
183      * @param adapterData Encoded adapter init data.
184      * @param strategyData Encoded strategy init data.
185      */
186     function deployAdapter(
187       IERC20 asset,
188       DeploymentArgs memory adapterData,
189       DeploymentArgs memory strategyData,
190       uint256 initialDeposit
191:    ) external canCreate returns (address adapter) {

/// @audit Missing: '@return'
276      * @dev Deploys `MultiRewardsStaking` based on the latest templateTemplateKey.
277      */
278:    function deployStaking(IERC20 asset) external canCreate returns (address) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L87-L97

```solidity
File: src/vault/Vault.sol

/// @audit Missing: '@param caller'
398       /// @return Maximum amount of underlying `asset` token that may be deposited for a given address. Delegates to adapter.
399:      function maxDeposit(address caller) public view returns (uint256) {

/// @audit Missing: '@param caller'
403       /// @return Maximum amount of vault shares that may be minted to given address. Delegates to adapter.
404:      function maxMint(address caller) external view returns (uint256) {

/// @audit Missing: '@param caller'
408       /// @return Maximum amount of underlying `asset` token that can be withdrawn by `caller` address. Delegates to adapter.
409:      function maxWithdraw(address caller) external view returns (uint256) {

/// @audit Missing: '@param caller'
413       /// @return Maximum amount of shares that may be redeemed by `caller` address. Delegates to adapter.
414:      function maxRedeem(address caller) external view returns (uint256) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L398-L399

### [N&#x2011;13]  Not using the named return variables anywhere in the function is confusing
Consider changing the variable to be an unnamed one

*There are 3 instances of this issue:*

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

/// @audit shares
380       function _convertToShares(uint256 assets, Math.Rounding rounding)
381           internal
382           view
383           virtual
384           override
385:          returns (uint256 shares)

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L380-L385

```solidity
File: src/vault/AdminProxy.sol

/// @audit success
/// @audit returndata
19      function execute(address target, bytes calldata callData)
20        external
21        onlyOwner
22:       returns (bool success, bytes memory returndata)

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/AdminProxy.sol#L19-L22

### [N&#x2011;14]  Consider using `delete` rather than assigning zero to clear values
The `delete` keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic

*There is 1 instance of this issue:*

```solidity
File: src/utils/MultiRewardStaking.sol

186:        accruedRewards[user][_rewardTokens[i]] = 0;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L186

### [N&#x2011;15]  Contracts should have full test coverage
While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.

*There is 1 instance of this issue:*

```solidity
File: Various Files


```

### [N&#x2011;16]  Large or complicated code bases should implement fuzzing tests
Large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts, should implement [fuzzing tests](https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05). Fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and fuzzers, with properly and extensively-written invariants, can close this testing gap significantly.

*There is 1 instance of this issue:*

```solidity
File: Various Files


```

### [N&#x2011;17]  Function ordering does not follow the Solidity style guide
According to the [Solidity style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions), functions should be laid out in the following order :`constructor()`, `receive()`, `fallback()`, `external`, `public`, `internal`, `private`, but the cases below do not follow this pattern

*There are 33 instances of this issue:*

```solidity
File: src/utils/MultiRewardEscrow.sol

/// @audit _getClaimableAmount() came earlier
207:    function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L207

```solidity
File: src/utils/MultiRewardStaking.sol

/// @audit decimals() came earlier
75:     function deposit(uint256 _amount) external returns (uint256) {

/// @audit _transfer() came earlier
170:    function claimRewards(address user, IERC20[] memory _rewardTokens) external accrueRewards(msg.sender, user) {

/// @audit _lockToken() came earlier
243     function addRewardToken(
244       IERC20 rewardToken,
245       uint160 rewardsPerSecond,
246       uint256 amount,
247       bool useEscrow,
248       uint192 escrowPercentage,
249       uint32 escrowDuration,
250       uint32 offset
251:    ) external onlyOwner {

/// @audit _calcRewardsEnd() came earlier
362:    function getAllRewardsTokens() external view returns (IERC20[] memory) {

/// @audit _accrueUser() came earlier
445     function permit(
446       address owner,
447       address spender,
448       uint256 value,
449       uint256 deadline,
450       uint8 v,
451       bytes32 r,
452:      bytes32 s

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L75

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

/// @audit __AdapterBase_init() came earlier
89        function decimals()
90            public
91            view
92            override(IERC20Metadata, ERC20)
93:           returns (uint8)

/// @audit _deposit() came earlier
173       function withdraw(
174           uint256 assets,
175           address receiver,
176           address owner
177:      ) public virtual override returns (uint256) {

/// @audit _withdraw() came earlier
247:      function totalAssets() public view override returns (uint256) {

/// @audit _underlyingBalance() came earlier
271       function convertToUnderlyingShares(uint256 assets, uint256 shares)
272           public
273           view
274           virtual
275:          returns (uint256)

/// @audit _previewDeposit() came earlier
304       function previewMint(uint256 shares)
305           public
306           view
307           virtual
308           override
309:          returns (uint256)

/// @audit _previewMint() came earlier
329       function previewWithdraw(uint256 assets)
330           public
331           view
332           virtual
333           override
334:          returns (uint256)

/// @audit _previewWithdraw() came earlier
353       function previewRedeem(uint256 shares)
354           public
355           view
356           virtual
357           override
358:          returns (uint256)

/// @audit _convertToShares() came earlier
404       function maxDeposit(address)
405           public
406           view
407           virtual
408           override
409:          returns (uint256)

/// @audit _verifyAndSetupStrategy() came earlier
500:      function setHarvestCooldown(uint256 newCooldown) external onlyOwner {

/// @audit setPerformanceFee() came earlier
574:      function pause() external onlyOwner {

/// @audit _protocolWithdraw() came earlier
610       function supportsInterface(bytes4 interfaceId)
611           public
612           view
613           virtual
614           override
615:          returns (bool)

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L89-L93

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

/// @audit _underlyingBalance() came earlier
122       function convertToUnderlyingShares(uint256, uint256 shares)
123           public
124           view
125           override
126:          returns (uint256)

/// @audit convertToUnderlyingShares() came earlier
136:      function rewardTokens() external view override returns (address[] memory) {

/// @audit _protocolWithdraw() came earlier
221:      function claim() public override onlyStrategy {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L122-L126

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

/// @audit _calculateLockedProfit() came earlier
129       function convertToUnderlyingShares(uint256, uint256 shares)
130           public
131           view
132           override
133:          returns (uint256)

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L129-L133

```solidity
File: src/vault/VaultController.sol

/// @audit _handleInitialDeposit() came earlier
186     function deployAdapter(
187       IERC20 asset,
188       DeploymentArgs memory adapterData,
189       DeploymentArgs memory strategyData,
190       uint256 initialDeposit
191:    ) external canCreate returns (address adapter) {

/// @audit _deployStrategy() came earlier
278:    function deployStaking(IERC20 asset) external canCreate returns (address) {

/// @audit _deployStaking() came earlier
313:    function proposeVaultAdapters(address[] calldata vaults, IERC4626[] calldata newAdapter) external {

/// @audit _registerVault() came earlier
408:    function setPermissions(address[] calldata targets, Permission[] calldata newPermissions) external onlyOwner {

/// @audit addStakingRewardsTokens() came earlier
483     function changeStakingRewardsSpeeds(
484       address[] calldata vaults,
485       IERC20[] calldata rewardTokens,
486:      uint160[] calldata rewardsSpeeds

/// @audit _verifyEqualArrayLength() came earlier
723:    function nominateNewAdminProxyOwner(address newOwner) external onlyOwner {

/// @audit _setDeploymentController() came earlier
864:    function setActiveTemplateId(bytes32 templateCategory, bytes32 templateId) external onlyOwner {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L186-L191

```solidity
File: src/vault/Vault.sol

/// @audit deposit() came earlier
160:      function mint(uint256 shares) external returns (uint256) {

/// @audit withdraw() came earlier
242:      function redeem(uint256 shares) external returns (uint256) {

/// @audit previewMint() came earlier
358       function previewWithdraw(uint256 assets)
359           external
360           view
361:          returns (uint256 shares)

/// @audit maxDeposit() came earlier
404:      function maxMint(address caller) external view returns (uint256) {

/// @audit accruedPerformanceFee() came earlier
473       function takeManagementAndPerformanceFees()
474           external
475           nonReentrant
476:          takeFees

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L160

### [N&#x2011;18]  Contract does not follow the Solidity style guide's suggested layout ordering
The [style guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout) says that, within a contract, the ordering should be 1) Type declarations, 2) State variables, 3) Events, 4) Modifiers, and 5) Functions, but the contract(s) below do not follow this ordering

*There are 35 instances of this issue:*

```solidity
File: src/utils/MultiRewardEscrow.sol

/// @audit function getEscrows came earlier
64      mapping(bytes32 => Escrow) public escrows;
65    
66      // User => Escrows
67      mapping(address => bytes32[]) public userEscrowIds;
68      // User => RewardsToken => Escrows
69:     mapping(address => mapping(IERC20 => bytes32[])) public userEscrowIdsByToken;

/// @audit function lock came earlier
136:    event RewardsClaimed(IERC20 indexed token, address indexed account, uint256 amount);

/// @audit function _getClaimableAmount came earlier
191:    address public feeRecipient;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L64-L69

```solidity
File: src/utils/MultiRewardStaking.sol

/// @audit function _transfer came earlier
157:    IMultiRewardEscrow public escrow;

/// @audit function _lockToken came earlier
208:    IERC20[] public rewardTokens;

/// @audit function getAllRewardsTokens came earlier
371     modifier accrueRewards(address _caller, address _receiver) {
372       IERC20[] memory _rewardTokens = rewardTokens;
373       for (uint8 i; i < _rewardTokens.length; i++) {
374         IERC20 rewardToken = _rewardTokens[i];
375         RewardInfo memory rewards = rewardInfos[rewardToken];
376   
377         if (rewards.rewardsPerSecond > 0) _accrueRewards(rewardToken, _accrueStatic(rewards));
378         _accrueUser(_receiver, rewardToken);
379   
380         // If a deposit/withdraw operation gets called for another user we should accrue for both of them to avoid potential issues like in the Convex-Vulnerability
381         if (_receiver != _caller) _accrueUser(_caller, rewardToken);
382       }
383       _;
384:    }

/// @audit function _accrueUser came earlier
438:    uint256 internal INITIAL_CHAIN_ID;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L157

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

/// @audit function _withdraw came earlier
241:      uint256 internal underlyingBalance;

/// @audit function maxMint came earlier
427:      IStrategy public strategy;

/// @audit function _verifyAndSetupStrategy came earlier
489:      uint256 public harvestCooldown;

/// @audit function setHarvestCooldown came earlier
513:      uint256 public performanceFee;

/// @audit function setPerformanceFee came earlier
559       modifier takeFees() {
560           _;
561   
562           uint256 fee = accruedPerformanceFee();
563           if (fee > 0) _mint(FEE_RECIPIENT, convertToShares(fee));
564   
565           uint256 shareValue = convertToAssets(1e18);
566           if (shareValue > highWaterMark) highWaterMark = shareValue;
567:      }

/// @audit function supportsInterface came earlier
625:      uint256 internal INITIAL_CHAIN_ID;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L241

```solidity
File: src/vault/CloneFactory.sol

/// @audit function constructor came earlier
29:     event Deployment(address indexed clone);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneFactory.sol#L29

```solidity
File: src/vault/CloneRegistry.sol

/// @audit function constructor came earlier
28:     mapping(address => bool) public cloneExists;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneRegistry.sol#L28

```solidity
File: src/vault/PermissionRegistry.sol

/// @audit function constructor came earlier
26:     mapping(address => Permission) public permissions;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L26

```solidity
File: src/vault/TemplateRegistry.sol

/// @audit function constructor came earlier
31      mapping(bytes32 => mapping(bytes32 => Template)) public templates;
32      mapping(bytes32 => bytes32[]) public templateIds;
33      mapping(bytes32 => bool) public templateExists;
34    
35:     mapping(bytes32 => bool) public templateCategoryExists;

/// @audit function addTemplate came earlier
88      event TemplateEndorsementToggled(
89        bytes32 templateCategory,
90        bytes32 templateId,
91        bool oldEndorsement,
92        bool newEndorsement
93:     );

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L31-L35

```solidity
File: src/vault/VaultController.sol

/// @audit function constructor came earlier
76:     event VaultDeployed(address indexed vault, address indexed staking, address indexed adapter);

/// @audit function changeVaultFees came earlier
387:    IVaultRegistry public vaultRegistry;

/// @audit function fundStakingRewards came earlier
535:    IMultiRewardEscrow public escrow;

/// @audit function _verifyEqualArrayLength came earlier
704     modifier canCreate() {
705       if (
706         permissionRegistry.endorsed(address(1))
707           ? !permissionRegistry.endorsed(msg.sender)
708           : permissionRegistry.rejected(msg.sender)
709       ) revert NotAllowed(msg.sender);
710       _;
711:    }

/// @audit modifier canCreate came earlier
717:    IAdminProxy public adminProxy;

/// @audit function acceptAdminProxyOwnership came earlier
739:    uint256 public performanceFee;

/// @audit function setAdapterPerformanceFees came earlier
779:    uint256 public harvestCooldown;

/// @audit function setAdapterHarvestCooldowns came earlier
819:    IDeploymentController public deploymentController;

/// @audit function _setDeploymentController came earlier
851:    mapping(bytes32 => bytes32) public activeTemplateId;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L76

```solidity
File: src/vault/VaultRegistry.sol

/// @audit function constructor came earlier
28      mapping(address => VaultMetadata) public metadata;
29    
30      // asset to vault addresses
31:     mapping(address => address[]) public vaultsByAsset;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L28-L31

```solidity
File: src/vault/Vault.sol

/// @audit function decimals came earlier
108       event Deposit(
109           address indexed caller,
110           address indexed owner,
111           uint256 assets,
112           uint256 shares
113:      );

/// @audit function accruedPerformanceFee came earlier
466:      uint256 public highWaterMark = 1e18;

/// @audit function takeManagementAndPerformanceFees came earlier
480       modifier takeFees() {
481           uint256 managementFee = accruedManagementFee();
482           uint256 totalFee = managementFee + accruedPerformanceFee();
483           uint256 currentAssets = totalAssets();
484           uint256 shareValue = convertToAssets(1e18);
485   
486           if (shareValue > highWaterMark) highWaterMark = shareValue;
487   
488           if (managementFee > 0) feesUpdatedAt = block.timestamp;
489   
490           if (totalFee > 0 && currentAssets > 0)
491               _mint(feeRecipient, convertToShares(totalFee));
492   
493           _;
494:      }

/// @audit modifier syncFeeCheckpoint came earlier
505:      VaultFees public fees;

/// @audit function setFeeRecipient came earlier
565:      IERC4626 public adapter;

/// @audit function changeAdapter came earlier
619:      uint256 public quitPeriod = 3 days;

/// @audit function unpause came earlier
657:      uint256 internal INITIAL_CHAIN_ID;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L108-L113

### [N&#x2011;19]  Interfaces should be indicated with an `I` prefix in the contract name

*There is 1 instance of this issue:*

```solidity
File: src/vault/adapter/yearn/IYearn.sol

8:    interface VaultAPI is IERC20 {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/IYearn.sol#L8


___

## Excluded findings
These findings are excluded from awards calculations because there are publicly-available automated tools that find them. The valid ones appear here for completeness

## Summary

### Low Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | Unsafe use of `transfer()`/`transferFrom()` with `IERC20` | 1 | 
| [L&#x2011;02] | Return values of `transfer()`/`transferFrom()` not checked | 1 | 
| [L&#x2011;03] | `safeApprove()` is deprecated | 1 | 
| [L&#x2011;04] | Missing checks for `address(0x0)` when assigning values to `address` state variables | 1 | 

Total: 4 instances over 4 issues


### Non-critical Issues
| |Issue|Instances|
|-|:-|:-:|
| [N&#x2011;01] | Return values of `approve()` not checked | 5 | 
| [N&#x2011;02] | `public` functions not called by the contract should be declared `external` instead | 4 | 
| [N&#x2011;03] | Event is missing `indexed` fields | 27 | 

Total: 36 instances over 3 issues





## Low Risk Issues

### [L&#x2011;01]  Unsafe use of `transfer()`/`transferFrom()` with `IERC20`
Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens.  For example Tether (USDT)'s `transfer()` and `transferFrom()` functions on L1 do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to `IERC20`, their [function signatures](https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca) do not match and therefore the calls made, revert (see [this](https://gist.github.com/IllIllI000/2b00a32e8f0559e8f386ea4f1800abc5) link for a test case). Use OpenZeppelinâ€™s `SafeERC20`'s `safeTransfer()`/`safeTransferFrom()` instead

*There is 1 instance of this issue:*

```solidity
File: src/vault/VaultController.sol

/// @audit (valid but excluded finding)
457:        IERC20(rewardsToken).transferFrom(msg.sender, address(adminProxy), amount);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L457

### [L&#x2011;02]  Return values of `transfer()`/`transferFrom()` not checked
Not all `IERC20` implementations `revert()` when there's a failure in `transfer()`/`transferFrom()`. The function signature has a `boolean` return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually making a payment

*There is 1 instance of this issue:*

```solidity
File: src/vault/VaultController.sol

/// @audit (valid but excluded finding)
457:        IERC20(rewardsToken).transferFrom(msg.sender, address(adminProxy), amount);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L457

### [L&#x2011;03]  `safeApprove()` is deprecated
[Deprecated](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/bfff03c0d2a59bcd8e2ead1da9aed9edf0080d05/contracts/token/ERC20/utils/SafeERC20.sol#L38-L45) in favor of `safeIncreaseAllowance()` and `safeDecreaseAllowance()`. If only setting the initial allowance to the value that means infinite, `safeIncreaseAllowance()` can be used instead. The function may currently work, but if a bug is found in this version of OpenZeppelin, and the version that you're forced to upgrade to no longer has this function, you'll encounter unnecessary delays in porting and testing replacement contracts.

*There is 1 instance of this issue:*

```solidity
File: src/utils/MultiRewardStaking.sol

/// @audit (valid but excluded finding)
271:        rewardToken.safeApprove(address(escrow), type(uint256).max);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L271

### [L&#x2011;04]  Missing checks for `address(0x0)` when assigning values to `address` state variables

*There is 1 instance of this issue:*

```solidity
File: src/utils/MultiRewardEscrow.sol

/// @audit (valid but excluded finding)
31:       feeRecipient = _feeRecipient;

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L31

## Non-critical Issues

### [N&#x2011;01]  Return values of `approve()` not checked
Not all `IERC20` implementations `revert()` when there's a failure in `approve()`. The function signature has a `boolean` return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

*There are 5 instances of this issue:*

```solidity
File: src/vault/adapter/beefy/BeefyAdapter.sol

/// @audit (valid but excluded finding)
80:           IERC20(asset()).approve(_beefyVault, type(uint256).max);

/// @audit (valid but excluded finding)
83:               IERC20(_beefyVault).approve(_beefyBooster, type(uint256).max);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L80

```solidity
File: src/vault/adapter/yearn/YearnAdapter.sol

/// @audit (valid but excluded finding)
54:           IERC20(_asset).approve(address(yVault), type(uint256).max);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L54

```solidity
File: src/vault/VaultController.sol

/// @audit (valid but excluded finding)
171:        asset.approve(address(target), initialDeposit);

/// @audit (valid but excluded finding)
456:        IERC20(rewardsToken).approve(staking, type(uint256).max);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L171

### [N&#x2011;02]  `public` functions not called by the contract should be declared `external` instead
Contracts [are allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents' functions and change the visibility from `external` to `public`.

*There are 4 instances of this issue:*

```solidity
File: src/vault/Vault.sol

/// @audit (valid but excluded finding)
323       function previewDeposit(uint256 assets)
324           public
325           view
326:          returns (uint256 shares)

/// @audit (valid but excluded finding)
340:      function previewMint(uint256 shares) public view returns (uint256 assets) {

/// @audit (valid but excluded finding)
380       function previewRedeem(uint256 shares)
381           public
382           view
383:          returns (uint256 assets)

/// @audit (valid but excluded finding)
399:      function maxDeposit(address caller) public view returns (uint256) {

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L323-L326

### [N&#x2011;03]  Event is missing `indexed` fields
Index event fields make the field more quickly accessible [to off-chain tools](https://ethereum.stackexchange.com/questions/40396/can-somebody-please-explain-the-concept-of-event-indexing) that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each `event` should use three `indexed` fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*There are 27 instances of this issue:*

```solidity
File: src/interfaces/vault/IERC4626.sol

/// @audit (valid but excluded finding)
8:      event Deposit(address indexed sender, address indexed owner, uint256 assets, uint256 shares);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/interfaces/vault/IERC4626.sol#L8

```solidity
File: src/utils/MultiRewardEscrow.sol

/// @audit (valid but excluded finding)
73:     event Locked(IERC20 indexed token, address indexed account, uint256 amount, uint32 duration, uint32 offset);

/// @audit (valid but excluded finding)
136:    event RewardsClaimed(IERC20 indexed token, address indexed account, uint256 amount);

/// @audit (valid but excluded finding)
196:    event FeeSet(IERC20 indexed token, uint256 amount);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L73

```solidity
File: src/utils/MultiRewardStaking.sol

/// @audit (valid but excluded finding)
159:    event RewardsClaimed(address indexed user, IERC20 rewardToken, uint256 amount, bool escrowed);

/// @audit (valid but excluded finding)
220:    event RewardInfoUpdate(IERC20 rewardToken, uint160 rewardsPerSecond, uint32 rewardsEndTimestamp);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L159

```solidity
File: src/vault/adapter/abstracts/AdapterBase.sol

/// @audit (valid but excluded finding)
491:      event HarvestCooldownChanged(uint256 oldCooldown, uint256 newCooldown);

/// @audit (valid but excluded finding)
519:      event PerformanceFeeChanged(uint256 oldFee, uint256 newFee);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L491

```solidity
File: src/vault/CloneRegistry.sol

/// @audit (valid but excluded finding)
33:     event CloneAdded(address clone);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/CloneRegistry.sol#L33

```solidity
File: src/vault/PermissionRegistry.sol

/// @audit (valid but excluded finding)
28:     event PermissionSet(address target, bool newEndorsement, bool newRejection);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/PermissionRegistry.sol#L28

```solidity
File: src/vault/TemplateRegistry.sol

/// @audit (valid but excluded finding)
38:     event TemplateCategoryAdded(bytes32 templateCategory);

/// @audit (valid but excluded finding)
39:     event TemplateAdded(bytes32 templateCategory, bytes32 templateId, address implementation);

/// @audit (valid but excluded finding)
40:     event TemplateUpdated(bytes32 templateCategory, bytes32 templateId);

/// @audit (valid but excluded finding)
88      event TemplateEndorsementToggled(
89        bytes32 templateCategory,
90        bytes32 templateId,
91        bool oldEndorsement,
92        bool newEndorsement
93:     );

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L38

```solidity
File: src/vault/VaultController.sol

/// @audit (valid but excluded finding)
741:    event PerformanceFeeChanged(uint256 oldFee, uint256 newFee);

/// @audit (valid but excluded finding)
781:    event HarvestCooldownChanged(uint256 oldCooldown, uint256 newCooldown);

/// @audit (valid but excluded finding)
824:    event DeploymentControllerChanged(address oldController, address newController);

/// @audit (valid but excluded finding)
853:    event ActiveTemplateIdChanged(bytes32 oldKey, bytes32 newKey);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L741

```solidity
File: src/vault/VaultRegistry.sol

/// @audit (valid but excluded finding)
36:     event VaultAdded(address vaultAddress, string metadataCID);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L36

```solidity
File: src/vault/Vault.sol

/// @audit (valid but excluded finding)
42:       event VaultInitialized(bytes32 contractName, address indexed asset);

/// @audit (valid but excluded finding)
108       event Deposit(
109           address indexed caller,
110           address indexed owner,
111           uint256 assets,
112           uint256 shares
113:      );

/// @audit (valid but excluded finding)
512:      event NewFeesProposed(VaultFees newFees, uint256 timestamp);

/// @audit (valid but excluded finding)
513:      event ChangedFees(VaultFees oldFees, VaultFees newFees);

/// @audit (valid but excluded finding)
514:      event FeeRecipientUpdated(address oldFeeRecipient, address newFeeRecipient);

/// @audit (valid but excluded finding)
569:      event NewAdapterProposed(IERC4626 newAdapter, uint256 timestamp);

/// @audit (valid but excluded finding)
570:      event ChangedAdapter(IERC4626 oldAdapter, IERC4626 newAdapter);

/// @audit (valid but excluded finding)
621:      event QuitPeriodSet(uint256 quitPeriod);

```
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L42
