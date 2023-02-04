## [YO GAS-1] Use assembly to check for address(0)

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L83
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L440
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L95
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L96


### Recommended Mitigation Steps
Use assembly to check for address(0)

## [YO GAS-2] USING BOOLS FOR STORAGE INCURS OVERHEAD

### Handle
yosuke

## Vulnerability details
### Impact
Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and  pointer aliasing, and it cannot be disabled.

### Proof of Concept
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/EIP165.sol#L7
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/WithRewards.sol#L21
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/AdminProxy.sol#L22
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneFactory.sol#L42
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L28
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L51
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L55
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L126
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L230
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L260
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L288
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L323
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L338
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L360
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L375
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L391
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L410
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L442
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L450
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L497
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L545
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L565
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L588
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L609
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L622
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L635
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L648
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L767
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L807
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L247
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L234
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L140
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L91
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L92
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L105

### Recommended Mitigation Steps
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past.

## [YO GAS-3] Use calldata instead of memory for function arguments that do not get mutated.

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L64
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L65
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L90
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L120
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L144
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L154
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L205
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L206
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L226
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L227
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L242
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L256
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L390
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L390
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L170
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/TemplateRegistry.sol#L70


### Recommended Mitigation Steps
Use calldata instead of memory for function arguments that do not get mutated because of lower gas fee.
ex)
before

```solidity=
Template memory template
```

after

```solidity=
Template calldata template
```

## [YO GAS-4] Using private rather than public for constants and immutable, saves gas

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L36-L39
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L25

### Recommended Mitigation Steps
ex)
before

```solidity=
uint256 constant DEGRADATION_COEFFICIENT = 10**18;
```

after

```solidity=
uint256 private constant DEGRADATION_COEFFICIENT = 10**18;
```

## [YO GAS-5] It is cheaper in gas to use "value!=0" instead of "value>0" for comparing variables of type uint.

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L563


### Recommended Mitigation Steps
ex)
before

```solidity=
if (fee > 0) _mint(FEE_RECIPIENT, convertToShares(fee));
```

after

```solidity=
if (fee != 0) _mint(FEE_RECIPIENT, convertToShares(fee));
```

## [YO GAS-6] USE FUNCTION INSTEAD OF MODIFIERS

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/OnlyStrategy.sol#L10


### Recommended Mitigation Steps
USE FUNCTION INSTEAD OF MODIFIERS because of lower gas fee.
ex)
before

```solidity=
modifier onlyStrategy() {
    if (msg.sender != address(this)) revert NotStrategy(msg.sender);
    _;
}
```

after

```solidity=
function onlyStrategy() private {
    if (msg.sender != address(this)) revert NotStrategy(msg.sender);
}
```

## [YO GAS-7] `++I`/`I++` SHOULD BE `UNCHECKED{++I}`/`UNCHECKED{I++}` WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN `FOR`AND `WHILE`LOOPS

### Handle
yosuke

## Vulnerability details
### Impact

The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**.

### Proof of Concept
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L42
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L319
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L337
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L357
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L374
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L437
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L494
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L523
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L564
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L587
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L607
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L620
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L633
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L646
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L766
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L806
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L171
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L373
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L53
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L155
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L210

### Recommended Mitigation Steps

```solidity
for (uint256 i; i < length;) {
	...
	unchecked**{++i}**
}
```

## [YO GAS-8] X = X + Y, X = X - Y, X = X * Y, X = X / Y ARE MORE EFFICIENT, THAN X += Y, X -= Y, X *= Y, X /= Y

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L158
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L225
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L357
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L408
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L431
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L178
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L110
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L162

### Recommended Mitigation Steps
ex)
before

```solidity=
amount -= fee;
```

after

```solidity=
amount = amount - fee;
```

## [YO GAS-9] USE bytes32 INSTEAD OF string

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L31
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L32
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L49
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L50
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L59
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L63
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L24
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L25
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L66
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L71
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L90
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L99
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L21
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L22
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L47
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L52
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L61
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L70
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultRegistry.sol#L36

### Recommended Mitigation Steps
USE bytes32 INSTEAD OF string because of lower gas fee.
ex)
before

```solidity=
string internal _name;
```

after

```solidity=
bytes32 internal _name;
```