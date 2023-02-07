### Gas-Optimizations Template
| Count | Explanation | Instances | Gas saved per instance | All |
|:--:|:-------:|:--:|:--:|:--:|
| G-01 | Use assembly to check if  `x` equals `y` in if statement | 4 | 24 | 96 |
| G-02 | Use assembly to check for address zero | 12 | 26 | 312 |
| G-03 | Use assembly to write address storage values | 5 | 39 | 195 |
| G-04 | Use assembly in the constructor | 1 | 23 | 23 |
| G-05 | if statement can be applied instead of creating an internal function | 9 | 33 | 297 |
| G-06 | Create a local variable as a holder instead of emitting state variables | 11 | 10 | 110 |
| G-07 | Mathematical operations which are impossible to overflow should be unchecked | 20 | 204 | 4080 |
| G-08 | Caching a mapping's value in a local storage variable and applying the x = x + y optimization on top of that saves gas | 2 | 22 | 44 |
| G-09 | Calling an internal function with cached value of state variable cost a decent amount of gas than just applying the state variable itself | 1 | 2 158 | 2 158 |
| G-10 | Save gas by breaking up an if statement with multiple conditions, into multiple if statements with a single condition | 10 | 14 | 140 |
| G-11 | The function `claimRewards` can be refactored to use storage pointer instead of memory | 1 | 96 | 96 |
| G-12 | State variables which don't change after deploying time should be set as immutable | 5 | 2 133 | 10 665 |
| G-13 | Pre-increments cost less gas than post-increments | 1 | 5 | 5 |

| Total Gas Saved | 18 221 |
|:--:|:-------:|

All of the instances above are accurate and tested on foundry, for the tests protocol's pragma version ^0.8.15 is used.
The optimizer was turned on and set as 10000 runs:
```
optimizer = true
optimizer_runs = 10000
```

## Use assembly to check if  `x` equals `y` in if statement
In the below instance tested with foundry, we are using assembly to check if rewardsPerSecond equals zero.
Which saves 12 gas per instance.

```js
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.15;

import "../../lib/test.sol";
import "../../lib/Console.sol";

contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;

    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }

    function testGas() public {
        c0.addRewardToken(1);
        c1.addRewardTokenAssembly(1);
    }
}

contract Contract0 {
 
    function addRewardToken(uint160 rewardsPerSecond) public {
        if (rewardsPerSecond == 0) revert ();
    }
}

contract Contract1 {

    function addRewardTokenAssembly(uint160 rewardsPerSecond) public {
        assembly {
            if eq(rewardsPerSecond, 0) { revert(rewardsPerSecond, 0) }
        }
    }
}
```

```js
| src/test/GasTest.t.sol:Contract0 contract |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| addRewardToken                            | 264             | 264 | 264    | 264 | 1       |


| src/test/GasTest.t.sol:Contract1 contract |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 37487                                     | 218             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| addRewardTokenAssembly                    | 252             | 252 | 252    | 252 | 1       |
```

Instances:
```solidity
src/utils/MultiRewardStaking.sol

174: if (rewardAmount == 0)
258: if (rewardsPerSecond == 0)
299: if (rewardsPerSecond == 0)
325: if (amount == 0)
420: if (oldIndex == 0) {

src/utils/MultiRewardEscrow.sol

97: if (amount == 0)
98: if (duration == 0)
160: if (claimable == 0)
```

## Use assembly to check for address zero
In the below instance assembly is used instead of if statement to check for address(0).
Which saves 26 gas per instance.

```js
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.15;

import "../../lib/test.sol";
import "../../lib/Console.sol";

contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;

    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }

    function testGas() public {
        c0.transfer(address(1), address(2));
        c1.transferAssembly(address(1), address(2));
    }
}

contract Contract0 {
 
    function transfer(
    address from,
    address to
  ) public {
    if (from == address(0) || to == address(0)) revert();
  }
}

contract Contract1 {

    function transferAssembly(
    address from,
    address to
  ) public {
    assembly {
        if iszero(from) { revert(from, 0x00) }
        if iszero(to) { revert(to, 0x00) }
    }
  }
}
```

```js
| src/test/GasTest.t.sol:Contract0 contract |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 53905                                     | 301             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| transfer                                  | 422             | 422 | 422    | 422 | 1       |


| src/test/GasTest.t.sol:Contract1 contract |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 44893                                     | 255             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| transferAssembly                          | 396             | 396 | 396    | 396 | 1       |
```

Instances:
```solidity
src/utils/MultiRewardStaking.sol

142: if (from == address(0) || to == address(0))

src/vault/Vault.sol

74: if (address(asset_) == address(0))
90: if (feeRecipient_ == address(0))
141: if (receiver == address(0))
177: if (receiver == address(0))
216: if (receiver == address(0))
258: if (receiver == address(0))
554: if (_feeRecipient == address(0))

src/vault/adapter/beefy/BeefyAdapter.sol

138: if (address(beefyBooster) == address(0))
222: if (address(beefyBooster) == address(0))

src/utils/MultiRewardEscrow.sol

95: if (token == IERC20(address(0)))
96: if (account == address(0))
```

## Use assembly to write address storage values
In the below instance assembly is used to write storage value for address.
Which saves 39 gas per instance.

```js
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.15;

import "../../lib/test.sol";
import "../../lib/Console.sol";

contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;

    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }

    function testGas() public {
        c0.setFeeRecipient(address(1));
        c1.setFeeRecipient(address(1));
    }
}

contract Contract0 {
    address public feeRecipient;

    function setFeeRecipient(address _feeRecipient) external {
    
        feeRecipient = _feeRecipient;
    }
}

contract Contract1 {
    address public feeRecipient;

    function setFeeRecipient(address _feeRecipient) external {
        assembly {
            sstore(feeRecipient.slot, _feeRecipient)
        }
    }
}
```

```js
| src/test/GasTest.t.sol:Contract0 contract |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 66917                                     | 366             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setFeeRecipient                           | 22385           | 22385 | 22385  | 22385 | 1       |


| src/test/GasTest.t.sol:Contract1 contract |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 51705                                     | 290             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setFeeRecipient                           | 22346           | 22346 | 22346  | 22346 | 1       |
```

Instances:
```solidity
src/vault/Vault.sol

77: asset = asset_;
78: adapter = adapter_;
91: feeRecipient = feeRecipient_;
553: function setFeeRecipient
578: function proposeAdapter
```

## Use assembly in the constructor
Use assembly when declaring storage values at a deploying time (in the constructor).
This saves 23 gas per instance.

```js
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.15;

import "../../lib/test.sol";
import "../../lib/Console.sol";

contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;

    function setUp() public {
        c0 = new Contract0(address(1));
        c1 = new Contract1(address(1));
    }

    function testGas() public {
        c0;
    }

    function testGas2() public {
        c1;
    }
}

contract Contract0 {
    address public feeRecipient;

    constructor(address _feeRecipient){
        feeRecipient = _feeRecipient;
    }

}

contract Contract1 {
    address public feeRecipient;

    constructor(address _feeRecipient){
        assembly {
            sstore(feeRecipient.slot, _feeRecipient)
        }
    }
}
```

```js
[PASS] testGas() (gas: 143)
[PASS] testGas2() (gas: 120)
```

Instances:
```solidity
src/utils/MultiRewardEscrow.sol

31: feeRecipient = _feeRecipient;
```

## if statement can be applied instead of creating an internal function
An internal function is created just to check if value x doesn't equal value y.
Consider deleting the function and just doing the check in the core functions.
This saves 33 gas per instance.

```solidity
700:  function _verifyEqualArrayLength(uint256 length1, uint256 length2) internal pure {
701:    if (length1 != length2) revert ArrayLengthMissmatch();
702:  }
```

```js
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.15;

import "../../lib/test.sol";
import "../../lib/Console.sol";

contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;

    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }

    function testGas() public {
        c0.first();
        c1.second();
    }
}

contract Contract0 {

    function first() public {
        _verifyEqualArrayLength(1, 1);
    }

    function _verifyEqualArrayLength(uint256 length1, uint256 length2) internal pure {
    if (length1 != length2) revert ();
  }
}

contract Contract1 {

    function second() public returns (uint256) {
        if (1 != 1) revert ();
    }
}
```

```js
| src/test/GasTest.t.sol:Contract0 contract |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 26681                                     | 163             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| first                                     | 179             | 179 | 179    | 179 | 1       |


| src/test/GasTest.t.sol:Contract1 contract |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 23875                                     | 148             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| second                                    | 146             | 146 | 146    | 146 | 1       |
```

Instances:
```solidity
src/vault/VaultController.sol

316: _verifyEqualArrayLength(len, newAdapter.length);
355: _verifyEqualArrayLength(len, fees.length);
434:  _verifyEqualArrayLength(vaults.length, rewardTokenData.length);
490: _verifyEqualArrayLength(len, rewardTokens.length);
491: _verifyEqualArrayLength(len, rewardsSpeeds.length);
519: _verifyEqualArrayLength(len, rewardTokens.length);
520: _verifyEqualArrayLength(len, amounts.length);
544: _verifyEqualArrayLength(tokens.length, fees.length);
584: _verifyEqualArrayLength(len, templateIds.length);
```

## Create a local variable as a holder instead of emitting state variables
In the example below, we create a local cache which will be emitted instead of applying the state variable.
This saves 10 gas per instance.

```js
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.15;

import "../../lib/test.sol";
import "../../lib/Console.sol";

contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;

    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }

    function testGas() public {
        c0.first();
        c1.second();
    }
}

contract Contract0 {

    event PerformanceFeeChanged(uint256 fee);

    uint256 public performanceFee;

    function first() public {
        emit PerformanceFeeChanged(performanceFee);
    }
}

contract Contract1 {

    event PerformanceFeeChanged(uint256 fee);

    uint256 public performanceFee;

    function second() public {
        uint256 _performanceFee = performanceFee;
        emit PerformanceFeeChanged(_performanceFee);
        }
}

```

```js
| src/test/GasTest.t.sol:Contract0 contract |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 40693                                     | 234             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| first                                     | 3294            | 3294 | 3294   | 3294 | 1       |


| src/test/GasTest.t.sol:Contract1 contract |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 40093                                     | 230             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| second                                    | 3284            | 3284 | 3284   | 3284 | 1       |
```

Instances:
```solidity
src/vault/VaultController.sol

755: emit PerformanceFeeChanged(performanceFee, newFee);
795: emit HarvestCooldownChanged(harvestCooldown, newCooldown);
840: emit DeploymentControllerChanged(address(deploymentController), address(_deploymentController));

src/vault/Vault.sol

97:  emit VaultInitialized(contractName, address(asset));
544: emit ChangedFees(fees, proposedFees);
556: emit FeeRecipientUpdated(feeRecipient, _feeRecipient);
585: emit NewAdapterProposed(newAdapter, block.timestamp);
606: emit ChangedAdapter(adapter, proposedAdapter);
635: emit QuitPeriodSet(quitPeriod);

src/vault/adapter/abstracts/AdapterBase.sol

504: emit HarvestCooldownChanged(harvestCooldown, newCooldown);
553: emit PerformanceFeeChanged(performanceFee, newFee);
```

## Mathematical operations which are impossible to overflow should be unchecked
Operations which are impossible to overflow, should be set as unchecked.
This successfuly saves 203 gas per instance.

```js
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.15;

import "../../lib/test.sol";
import "../../lib/Console.sol";

contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;

    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }

    function testGas() public {
        c0.first();
        c1.second();
    }
}

contract Contract0 {

    function first() public {
        for(uint256 i; i < 3; i++){

        }
    }
}

contract Contract1 {

    function second() public {
        for(uint256 i; i < 3;){
            unchecked { i++; }
        }
    }
    
}
```

```js
| src/test/GasTest.t.sol:Contract0 contract |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| first                                     | 487             | 487 | 487    | 487 | 1       |


| src/test/GasTest.t.sol:Contract1 contract |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 25675                                     | 158             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| second                                    | 283             | 283 | 283    | 283 | 1       |
```

Instances:
```solidity
src/vault/VaultController.sol

319: for (uint8 i = 0; i < len; i++) {
337: for (uint8 i = 0; i < len; i++) {
357: for (uint8 i = 0; i < len; i++) {
374: for (uint8 i = 0; i < len; i++) {
437: for (uint256 i = 0; i < len; i++) {
494: for (uint256 i = 0; i < len; i++) {
523: for (uint256 i = 0; i < len; i++) {
564: for (uint256 i = 0; i < len; i++) {
587: for (uint256 i = 0; i < len; i++) {
607: for (uint256 i = 0; i < len; i++) {
620: for (uint256 i = 0; i < len; i++) {
633: for (uint256 i = 0; i < len; i++) {
646: for (uint256 i = 0; i < len; i++) {
766: for (uint256 i = 0; i < len; i++) {
806: for (uint256 i = 0; i < len; i++) {

src/utils/MultiRewardStaking.sol

171:  for (uint8 i; i < _rewardTokens.length; i++) {
373: for (uint8 i; i < _rewardTokens.length; i++) {

src/utils/MultiRewardEscrow.sol
53: for (uint256 i = 0; i < escrowIds.length; i++) {
155: for (uint256 i = 0; i < escrowIds.length; i++) {
210: for (uint256 i = 0; i < tokens.length; i++) {
```

## Caching a mapping's value in a local storage variable and applying the x = x + y optimization on top of that saves gas
In the below example, we are caching the mapping's value into storage and applying the x = x + y gas optimization on top of that. This saves 22 gas per instance.

```js
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.15;

import "../../lib/test.sol";
import "../../lib/Console.sol";

contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;

    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }

    function testGas() public {
        c0.first(1);
        c1.second(1);
    }
}

contract Contract0 {

    struct RewardInfo {
        uint prize;
    }

     mapping(uint => RewardInfo) public rewardInfos;

    function first(uint _input) public {
        uint cache;

        rewardInfos[_input].prize += cache;
        

        }
    }

contract Contract1 {
    struct RewardInfo {
        uint prize;
    }

     mapping(uint => RewardInfo) public rewardInfos;

    function second(uint _input) public {
        uint cache;

        RewardInfo storage rewardInfo = rewardInfos[_input];

        rewardInfo.prize = rewardInfo.prize + cache;
        

        }
    }
```

```js
| src/test/GasTest.t.sol:Contract0 contract |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 59911                                     | 331             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| first                                     | 2581            | 2581 | 2581   | 2581 | 1       |


| src/test/GasTest.t.sol:Contract1 contract |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 58311                                     | 323             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| second                                    | 2559            | 2559 | 2559   | 2559 | 1       |
```

Instances:
```solidity
src/utils/MultiRewardStaking.sol

408: rewardInfos[_rewardToken].index += deltaIndex;

src/utils/MultiRewardEscrow.sol

162: escrows[escrowId].balance -= claimable;
```

## Calling an internal function with cached value of state variable cost a decent amount of gas than just applying the state variable itself
In the below example an internal function is called with a cached state variable, as a result this cost a decent amount of gas.
This saves 2158 gas per instance.

```js
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.15;

import "../../lib/test.sol";
import "../../lib/Console.sol";

contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;

    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }

    function testGas() public {
        c0.first();
        c1.second();
    }
}

contract Contract0 {

    uint256 public balance;

    
    function first() public {

        uint256 cache = balance;

        _add(cache);

        }

    function _add(uint256 amount) internal {}

    }

contract Contract1 {

    uint256 public balance;

    function second() public {

        _add(balance);

        }

    function _add(uint256 amount) internal {}

    }
```

```js
| src/test/GasTest.t.sol:Contract0 contract |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 31081                                     | 185             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| first                                     | 2256            | 2256 | 2256   | 2256 | 1       |


| src/test/GasTest.t.sol:Contract1 contract |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 27481                                     | 167             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| second                                    | 98              | 98  | 98     | 98  | 1       |
```

Instance:
```solidity
src/vault/adapter/abstracts/AdapterBase.sol

L536-L540 - cache of highWaterMark is used to call an internal function
```

## Save gas by breaking up an if statement with multiple conditions, into multiple if statements with a single condition
Breaking up if statements with multiple conditions saves 14 gas per instance.

```js
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.15;

import "../../lib/test.sol";
import "../../lib/Console.sol";

contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;

    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }

    function testGas() public {
        c0.first();
        c1.first();
    }
}

contract Contract0 {

  uint256 public value;
  uint256 public amount;

    function first() public returns (uint){

      if(value != 0 || amount != 0) return 3;

    }
  
}

contract Contract1 {

  uint256 public value;
  uint256 public amount;

    function first() public returns (uint){

      if(value != 0) return 3;
      if(amount != 0) return 3;

    }
}
```

```js
| src/test/GasTest.t.sol:Contract0 contract |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 37887                                     | 220             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| first                                     | 4427            | 4427 | 4427   | 4427 | 1       |


| src/test/GasTest.t.sol:Contract1 contract |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 37887                                     | 220             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| first                                     | 4413            | 4413 | 4413   | 4413 | 1       |
```

Instances:
```solidity
src/utils/MultiRewardEscrow.sol

L171-L176

src/vault/adapter/abstracts/AdapterBase.sol

670: if (recoveredAddress == address(0) || recoveredAddress != owner)

src/vault/VaultController.sol

669: if (msg.sender != metadata.creator || msg.sender != owner)
837: if (address(_deploymentController) == address(0) || address(deploymentController) == address(_deploymentController))

src/vault/Vault.sol

490: if (totalFee > 0 && currentAssets > 0)
526: if (
630: if (_quitPeriod < 1 days || _quitPeriod > 7 days)
702: if (recoveredAddress == address(0) || recoveredAddress != owner)

src/utils/MultiRewardStaking.sol

142: if (from == address(0) || to == address(0))
481: if (recoveredAddress == address(0) || recoveredAddress != owner)
```

## The function `claimRewards` can be refactored to use storage pointer instead of memory
As described the function `claimRewards` can be refactored to use storage pointer, which saves 96 per instance.

```js
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.15;

import "../../lib/test.sol";
import "../../lib/Console.sol";

contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;

    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }

    function testGas() public {
        c0.first(1);
        c1.first(1);
    }
}

contract Contract0 {

  struct Escrow {
    uint256 balance;
    uint256 lastUpdateTime;
  }

  mapping (uint => Escrow) public rewardInfo;

    function first(uint256 _input) public {

      Escrow memory escrow = rewardInfo[_input];

      _getClaimableAmount(escrow);

      rewardInfo[_input].balance = 2;
      rewardInfo[_input].lastUpdateTime = block.timestamp;

    }

    function _getClaimableAmount(Escrow memory escrow) internal view returns (uint256) {
    }
}


contract Contract1 {

  struct Escrow {
    uint256 balance;
    uint256 lastUpdateTime;
  }

  mapping (uint => Escrow) public rewardInfo;

    function first(uint256 _input) public {

      Escrow storage escrow = rewardInfo[_input];

      _getClaimableAmount(escrow);

      escrow.balance = 2;
      escrow.lastUpdateTime = block.timestamp;

    }

    function _getClaimableAmount(Escrow memory escrow) internal view returns (uint256) {
    }

}
```

```js
| src/test/GasTest.t.sol:Contract0 contract |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 56705                                     | 315             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| first                                     | 44651           | 44651 | 44651  | 44651 | 1       |


| src/test/GasTest.t.sol:Contract1 contract |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 52105                                     | 292             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| first                                     | 44555           | 44555 | 44555  | 44555 | 1       |
```

Instance:
```solidity
src/utils/MultiRewardEscrow.sol

154: function claimRewards
```

## State variables which don't change after deploying time should be set as immutable 
Tested this instance on remix, since l couldn't on foundry.
This saves 2133 gas per instance.

```js
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.15;

contract Contract0 {
    address public owner; //23553
    
    constructor(address _owner){  
        owner = _owner;
    }

}

contract Contract1 {
    address public immutable owner; //21420

    constructor(address _owner){  
        owner = _owner;
    }
}
```

Instance:
```solidity
src/vault/VaultController.sol

387: IVaultRegistry public vaultRegistry;
535: IMultiRewardEscrow public escrow;
717: IAdminProxy public adminProxy;
822: IPermissionRegistry public permissionRegistry;

src/utils/MultiRewardEscrow.sol

191: address public feeRecipient;
```

## Pre-increments cost less gas than post-increments
In the example below pre increment is used, which saves 5 gas per instance.

```js
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.15;

import "../../lib/test.sol";
import "../../lib/Console.sol";

contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;

    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }

    function testGas() public {
        c0.first();
        c1.first();
    }
}

contract Contract0 {

  uint256 public nonce;

  function first() public {
    nonce++;
  }
}


contract Contract1 {

  uint256 public nonce;

  function first() public {
    ++nonce;
  }
}
```

```js
| src/test/GasTest.t.sol:Contract0 contract |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 51505                                     | 289             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| first                                     | 22308           | 22308 | 22308  | 22308 | 1       |


| src/test/GasTest.t.sol:Contract1 contract |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 51099                                     | 286             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| first                                     | 22303           | 22303 | 22303  | 22303 | 1       |
```

Instances:
```solidity
src/utils/MultiRewardEscrow.sol

102: nonce++;
```