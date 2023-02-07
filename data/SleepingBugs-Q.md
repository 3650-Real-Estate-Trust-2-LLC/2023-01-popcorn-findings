



# Low

| | Issue index |
| ----------- | ----------- |
| 1 | [`initialize()` function can be called anybody](#`initialize()`-function-can-be-called-anybody) |
| 2 | [Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions](#upgradeable-contract-is-missing-a-`__gap[50]`-storage-variable-to-allow-for-new-storage-variables-in-later-versions) |
| 3 | [Return value not being checked](#return-value-not-being-checked) |

## `initialize()` function can be called anybody

`initialize()` function can be called anybody when the contract is not initialized. Therefore it can be front-runned.

The attacker can initialize the contract before the legitimate deployer, hoping that the victim continues to use the same contract.

In the best case for the victim, they notice it and have to redeploy their contract costing gas.

More importantly, if someone else runs this function, they will have full authority because of the `__Ownable_init()` function. 

### Code Snippets

`initialize()` function not mentioned to be called by factory:

- [MultiRewardStaking.sol#L41](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L41)

Mentioned to be called by factory:

- [YearnAdapter.sol#L34](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L34)

- [BeefyAdapter.sol#L46](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L46)

- [Vault.sol#L57](https://github.com/code-423n4/2023-01-popcorn/blob/7a513a9734b9e49af33041e2032ffc131f3b73b0/src/vault/Vault.sol#L57)


### Recommendation

Add a control that makes `initialize()` only call by the Deployer Contract;

```diff
+ if (msg.sender != DEPLOYER_ADDRESS) {revert NotDeployer();}
```

Use the constructor to initialize non-proxied contracts.

For initializing proxy contracts deploy contracts using a factory contract that immediately calls initialize after deployment or make sure to call it immediately after deployment and verify the transaction succeeded.

## Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions
### Impact 

For upgradeable contracts, there must be storage gap to "allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments" (quote OpenZeppelin). Otherwise it may be very difficult to write new implementation code. Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract. This could have unintended and very serious consequences to the child contracts, potentially causing loss of user fund or cause the contract to malfunction completely. [See](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps).

For a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

### Vulnerability Details

In the following context of the upgradeable contracts they are expected to use gaps for avoiding collision:

- `ERC20Upgradeable`
- `ERC4626Upgradeable`
- `OwnedUpgradeable`
- `PausableUpgradeable`
- `ReentrancyGuardUpgradeable`

However, none of these contracts contain storage gap. The storage gap is essential for upgradeable contract because "It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments". Refer to the bottom part of this [article](https://docs.openzeppelin.com/contracts/4.x/upgradeable).

If a contract inheriting from a base contract contains additional variable, then the base contract cannot be upgraded to include any additional variable, because it would overwrite the variable declared in its child contract. This greatly limits contract upgradeability.

### Code Snippet


- [MultiRewardStaking.sol#L26](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L26)
contract MultiRewardStaking is ERC4626Upgradeable, OwnedUpgradeable {

- [Vault.sol#L27](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L27)
    ERC20Upgradeable,
    ReentrancyGuardUpgradeable,
    PausableUpgradeable,
    OwnedUpgradeable

- [AdapterBase.sol#L28](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L28)
    ERC4626Upgradeable,
    PausableUpgradeable,
    OwnedUpgradeable,
    ReentrancyGuardUpgradeable,

### Tools Used

Manual analysis

### Recommendation

Recommend adding appropriate storage gap at the end of upgradeable contracts such as the below. 

`uint256[50] private __gap;`

Reference OpenZeppelin [upgradeable contract templates](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable).


## Return value not being checked 
### Details

Return values not being checked may lead into unexpected behaviors with functions. Not events/Error are being emitted if that fails, so functions would be called even of not being working as expect.

### Code Snippet

`deposit`
- [YearnAdapter.sol#L163](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L163)
- [VaultController.sol#L172](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L172)
- [Vault.sol#L195](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L195)
- [Vault.sol#L155](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L155)
- [Vault.sol#L612](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L612)

`redeem`
- [Vault.sol#L598](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L598-L602)

`withdraw`
- [YearnAdapter.sol#L171](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L171)
- [Vault.sol#L237](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L237)
- [Vault.sol#L275](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L275)

`rejected`
- [VaultController.sol#L706-L708](https://github.com/code-423n4/2023-01-popcorn/blob/d95VaultController.sol#L706fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L706-L708)

### Recommendation

Check values and `revert`/`emit` events if needed


# Informational

| | Issue index |
| ----------- | ----------- |
| 1 | [Missing inheritance](#missing-inheritance) |
| 2 | [Constants should be defined rather than using magic numbers](#constants-should-be-defined-rather-than-using-magic-numbers) |
| 3 | [Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10 ** 18`)](#use-scientific-notation-(e.g.-`1e18`)-rather-than-exponentiation-(e.g.-`10-**-18`)) |
| 4 | [Inconsistent spacing in comments](#inconsistent-spacing-in-comments) |
| 5 | [No same value input control](#no-same-value-input-control) |
| 6 | [Maximum line length exceeded](#maximum-line-length-exceeded) |
## Missing inheritance

Vault should inherit from IPausable

- [Vault.sol#L26-L730](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L26-L730)


## Constants should be defined rather than using magic numbers

Use of `1e18`, `1e17`, `2e17`... can be declared as constants

- [VaultController.sol#L753](https://github.com/code-423n4/2023-01-popcorn/blob/7a513a9734b9e49af33041e2032ffc131f3b73b0/src/vault/VaultController.sol#L753)

- [AdapterBase.sol#L551](https://github.com/code-423n4/2023-01-popcorn/blob/dcdd3ceda3d5bd87105e691ebc054fb8b04ae583/src/vault/adapter/abstracts/AdapterBase.sol#L551)

- [MultiRewardEscrow.sol#L108](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L108)
      uint256 fee = Math.mulDiv(amount, feePerc, 1e18);

- [MultiRewardEscrow.sol#L211](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L211)
      if (tokenFees[i] >= 1e17) revert DontGetGreedy(tokenFees[i]);

- [MultiRewardStaking.sol#L197](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L197)
    uint256 escrowed = rewardAmount.mulDiv(uint256(escrowInfo.escrowPercentage), 1e18, Math.Rounding.Down);

- [Vault.sol#L144](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L144)
            assets.mulDiv(uint256(fees.deposit), 1e18, Math.Rounding.Down)

- [Vault.sol#L183](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L183)
            1e18 - depositFee,

- [Vault.sol#L224](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L224)
            1e18 - withdrawalFee,

- [Vault.sol#L265](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L265)
            1e18,

- [Vault.sol#L330](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L330)
                assets.mulDiv(uint256(fees.deposit), 1e18, Math.Rounding.Down)

- [Vault.sol#L345](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L345)
            1e18 - depositFee,

- [Vault.sol#L367](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L367)
            1e18 - withdrawalFee,

- [Vault.sol#L389](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L389)
            1e18,

- [Vault.sol#L437](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L437)
                ) / 1e18

- [Vault.sol#L449](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L449)
        uint256 shareValue = convertToAssets(1e18);

- [Vault.sol#L456](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L456)
                    1e36,

- [Vault.sol#L466](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L466)
    uint256 public highWaterMark = 1e18;

- [Vault.sol#L484](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L484)
        uint256 shareValue = convertToAssets(1e18);

- [Vault.sol#L498](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L498)
        highWaterMark = convertToAssets(1e18);

- [Vault.sol#L527](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L527)
            newFees.deposit >= 1e18 ||

- [Vault.sol#L528](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L528)
            newFees.withdrawal >= 1e18 ||

- [Vault.sol#L529](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L529)
            newFees.management >= 1e18 ||

- [Vault.sol#L530](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L530)
            newFees.performance >= 1e18

- [AdapterBase.sol#L85](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L85)
        highWaterMark = 1e18;

- [AdapterBase.sol#L531](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L531)
        uint256 shareValue = convertToAssets(1e18);

- [AdapterBase.sol#L538](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L538)
                    1e36,

- [AdapterBase.sol#L565](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L565)
        uint256 shareValue = convertToAssets(1e18);



## Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10 ** 18`)

- [YearnAdapter.sol#L25](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L25)
    uint256 constant DEGRADATION_COEFFICIENT = 10**18;

- [MultiRewardStaking.sol#L274](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L274)
    uint64 ONE = (10**IERC20Metadata(address(rewardToken)).decimals()).safeCastTo64();

- [MultiRewardStaking.sol#L406](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L406)
      deltaIndex = accrued.mulDiv(uint256(10**decimals()), supplyTokens, Math.Rounding.Down).safeCastTo224();

### Recommendation

Consider replacing `10 ** value` notation to scientific notation

## Inconsistent spacing in comments

Some lines use `// x` and some use `//x`. Following the common style comment should be as follows:

```diff
-uint256 public genericDeclaration; //generic comment without space`
+uint256 public genericDeclaration; // generic comment with space`
```

- [IBeefy.sol#L17](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/IBeefy.sol#L17)
  //Returns total balance of underlying token in the vault and its strategies






## No same value input control

This is done in other fragments of the code but not here

-[Vault.sol#L553](https://github.com/code-423n4/2023-01-popcorn/blob/7a513a9734b9e49af33041e2032ffc131f3b73b0/src/vault/Vault.sol#L553)

-[Vault.sol#L629-L636](https://github.com/code-423n4/2023-01-popcorn/blob/7a513a9734b9e49af33041e2032ffc131f3b73b0/src/vault/Vault.sol#L629-L636)

-[VaultController.sol#L751-L758](https://github.com/code-423n4/2023-01-popcorn/blob/7a513a9734b9e49af33041e2032ffc131f3b73b0/src/vault/VaultController.sol#L751-L758)

-[VaultController.sol#L791-L798](https://github.com/code-423n4/2023-01-popcorn/blob/7a513a9734b9e49af33041e2032ffc131f3b73b0/src/vault/VaultController.sol#L791-L798)

-[AdapterBase.sol#L500-L506](https://github.com/code-423n4/2023-01-popcorn/blob/dcdd3ceda3d5bd87105e691ebc054fb8b04ae583/src/vault/adapter/abstracts/AdapterBase.sol#L500-L506)

-[AdapterBase.sol#L549-L556](https://github.com/code-423n4/2023-01-popcorn/blob/dcdd3ceda3d5bd87105e691ebc054fb8b04ae583/src/vault/adapter/abstracts/AdapterBase.sol#L549-L556)

### Recommendation

Add code like this:

`if (_newCoolAddress == newCoolAddress) revert ADDRESS_SAME();`


## Maximum line length exceeded

Usually lines in source code are limited to 80 characters. Today’s screens are much larger so it’s reasonable to stretch this in some cases. Solidity newer guidelines suggest 120 characters. 
Reference: Long lines should be wrapped to conform with Solidity Style [guidelines](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#maximum-line-length)

Following lines with more than 120:
 

- [AdminProxy.sol#L13](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/AdminProxy.sol#L13)

- [VaultRegistry.sol#L41](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultRegistry.sol#L41)

- [DeploymentController.sol#L91](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/DeploymentController.sol#L91)

- [TemplateRegistry.sol#L49](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/TemplateRegistry.sol#L49)

- [YearnAdapter.sol#L6](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/yearn/YearnAdapter.sol#L6)

- [MultiRewardEscrow.sol#L6](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L6)

- [MultiRewardEscrow.sol#L49](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardEscrow.sol#L49)

- [BeefyAdapter.sol#L6](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L6)

- [BeefyAdapter.sol#L16](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/beefy/BeefyAdapter.sol#L16)

- [MultiRewardStaking.sol#L6](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L6)

- [MultiRewardStaking.sol#L7](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L7)

- [MultiRewardStaking.sol#L22](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L22)

- [MultiRewardStaking.sol#L106](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L106)

- [MultiRewardStaking.sol#L120](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L120)

- [MultiRewardStaking.sol#L136](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L136)

- [MultiRewardStaking.sol#L291](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L291)

- [MultiRewardStaking.sol#L380](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/utils/MultiRewardStaking.sol#L380)

- [Vault.sol#L6](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L6)

- [Vault.sol#L55](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L55)

- [Vault.sol#L165](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L165)

- [Vault.sol#L398](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L398)

- [Vault.sol#L408](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L408)

- [Vault.sol#L425](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L425)

- [Vault.sol#L426](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L426)

- [Vault.sol#L686](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L686)

- [VaultController.sol#L5](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L5)

- [VaultController.sol#L87](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L87)

- [VaultController.sol#L477](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L477)

- [VaultController.sol#L604](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L604)

- [VaultController.sol#L630](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L630)

- [VaultController.sol#L837](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L837)

- [AdapterBase.sol#L6](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L6)

- [AdapterBase.sol#L7](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L7)

- [AdapterBase.sol#L52](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L52)

- [AdapterBase.sol#L262](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L262)

- [AdapterBase.sol#L269](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L269)

- [AdapterBase.sol#L436](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L436)

- [AdapterBase.sol#L465](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L465)

- [AdapterBase.sol#L654](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L654)
