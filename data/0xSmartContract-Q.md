## Summary
### Low Risk Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|[L-01]|ERC4626 does not work with fee-on-transfer tokens in project|  |
|[L-02]|Use ERC-5143: Slippage Protection for Tokenized Vault|  |
|[L-03]|initialize() function can be called by anybody | 1 |
|[L-04]|Insufficient coverage| All Contracts |
|[L-05]|Missing Event for initialize| 14 |
|[L-06]|Project Upgrade and Stop Scenario should be|  |
|[L-07]|Loss of precision due to rounding in ` amount / uint256(rewardsPerSecond) ` | 1 |
|[L-08]|Use Fuzzing Test for complicated code bases |  |
|[L-09]|Add to Event-Emit for critical function| 1 |
|[L-10]|The `Timestamp` variable is limited to work until 2106| 1 |
|[L-11]|Signature Malleability of EVM's ecrecover()| 3 |
|[L-12]|Update codes to avoid Compile Errors| 15 |
|[L-13]|`_withdraw` event is missing parameters| 1 |
|[L-14]|Critical Address Changes Should Use Two-step Procedure| 1 |
|[L-15]|Lack of Input Validation| 1 |
|[L-16]|Prevent division by 0| 1 |

Total 16 issues


### Non-Critical Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [N-01] |Omissions in Events|1|
| [N-02] |NatSpec comments should be increased in contracts|All Contracts|
| [N-03] |You should leave a space after the word "Popcorn Yearn" when concatenating with `string.concat`| 1 |
| [N-04] |`Function writing` that does not comply with the `Solidity Style Guide`| All Contracts |
| [N-05] |Include return parameters in NatSpec comments|All Contracts |
| [N-06] |Tokens accidentally sent to the contract cannot be recovered| 1 |
| [N-07] |Take advantage of Custom Error's return value property| 22 |
| [N-08] |Repeated validation logic can be converted into a function modifier| 5 |
| [N-09] |Use a more recent version of Solidity| 35 |
| [N-10] |For functions, follow Solidity standard naming conventions (internal function style rule)| 4  |
| [N-11] |Floating pragma|35 |
| [N-12] |Use descriptive names for Contracts and Libraries | All Contracts |
| [N-13] |Use SMTChecker |  |
| [N-14] |Add NatSpec Mapping comment| 8 |
| [N-15] |Showing the actual values of numbers in NatSpec comments makes checking and reading code easier| 6 |
| [N-16] |Lines are too long| 7 |
| [N-17] |Open TODOs| 1 |
| [N-18] |Constants on the left are better| 9 |
| [N-19] |Confusing event-emit argument names| 4 |
| [N-20] |`constants` should be defined rather than using magic numbers| 1|
| [N-21] |Missing timelock for critical parameter change| 1|
| [N-22] |Use the `delete` keyword instead of assigning a value of `0`| 2 |
| [N-23] |Variable names that consist of all capital letters should be reserved for only `constant/immutable` variables| 6 |
| [N-24] |Avoid _shadowing_ `inherited state variables`| 1 |
| [N-25] |Not using the named return variables anywhere in the function is confusing| 1 |
| [N-26] |Unnecessary Emission of block.timestamp| 2 |
| [N-27] |Function Calls in Loop Could Lead to Denial of Service due to Array length not being checked| 1 |
| [N-28] |Don't use _msgSender() if not supporting EIP-2771| 4 |

Total 28 issues


### [L-01] ERC4626 does not work with fee-on-transfer tokens in project



### Impact
ERC20 token contract can be deposited with the `deposit` function.

With the following part of the code, the ERC20 transfer from msg.sender to the contract takes place;
`asset.safeTransferFrom(msg.sender, address(this), assets);`

```solidity
src/vault/Vault.sol:
  134:     function deposit(uint256 assets, address receiver)
  135:         public
  136:         nonReentrant
  137:         whenNotPaused
  138:         syncFeeCheckpoint
  139:         returns (uint256 shares)
  140:     {
  141:         if (receiver == address(0)) revert InvalidReceiver();
  142: 
  143:         uint256 feeShares = convertToShares(
  144:             assets.mulDiv(uint256(fees.deposit), 1e18, Math.Rounding.Down)
  145:         );
  146: 
  147:         shares = convertToShares(assets) - feeShares;
  148: 
  149:         if (feeShares > 0) _mint(feeRecipient, feeShares);
  150: 
  151:         _mint(receiver, shares);
  152: 
  153:         asset.safeTransferFrom(msg.sender, address(this), assets);
  154: 
  155:         adapter.deposit(assets, address(this));
  156: 
  157:         emit Deposit(msg.sender, receiver, assets, shares);
  158:     }


```

### Proof Of Concept

Contract calls transfer from contractA 100 tokens to current contract
Current contract thinks it received 100 tokens
It updates `tokenBalances` sheet +100 tokens
While actually contract received only 90 tokens
That breaks whole math for given token



### Recommended Mitigation Steps

There are several possible scenarios to prevent that.

Check how much contract balance is increased/decreased after every transfer (costs more gas)
Make a separate contract that checks if the token has fee-on-transfer and store bool value depending on the result.
If there is fee-on-transfer you can throw a require not allowing to use such token in the system while still saving lots of gas checking it only once.
Or if you still want to allow fee-on-transfer tokens, amount variable must be updated to the balance difference after and before transfer.


This code block should be used if you want to ban fee-on-transfer tokens;

```solidity
uint256 balanceBefore = IERC20(token).balanceOf(address(this));
IERC20(token). safeTransferFrom(msg.sender, address(this), amount);
uint256 balanceAfter = IERC20(token).balanceOf(address(this));
require(balanceAfter - amount == balanceBefore, "feeOnTransfer token not supported");

```
### [L-02] Use ERC-5143: Slippage Protection for Tokenized Vault

Project use ERC-4626 standart

EIP-4626 is vulnerable to the so-called inflation attacks. This attack results from the possibility to manipulate the exchange rate and front run a victim’s deposit when the vault has low liquidity volume.


**Proposed mitigation**


The following standard extends the EIP-4626 Tokenized Vault standard with functions dedicated to the safe interaction between EOAs and the vault when price is subject to slippage.

https://eips.ethereum.org/EIPS/eip-5143

### [L-03] initialize() function can be called by anybody

`initialize()` function can be called anybody when the contract is not initialized.

More importantly, if someone else runs this function, they will have full authority because of the `__Ownable_init()` function.

Here is a definition of `initialize()` function.

```solidity

src/utils/MultiRewardStaking.sol:
  41:   function initialize(
  42:     IERC20 _stakingToken,
  43:     IMultiRewardEscrow _escrow,
  44:     address _owner
  45:   ) external initializer {
  46:     __ERC4626_init(IERC20Metadata(address(_stakingToken)));
  47:     __Owned_init(_owner);
  48: 
  49:     _name = string(abi.encodePacked("Staked ", IERC20Metadata(address(_stakingToken)).name()));
  50:     _symbol = string(abi.encodePacked("pst-", IERC20Metadata(address(_stakingToken)).symbol()));
  51:     _decimals = IERC20Metadata(address(_stakingToken)).decimals();
  52: 
  53:     escrow = _escrow;
  54: 
  55:     INITIAL_CHAIN_ID = block.chainid;
  56:     INITIAL_DOMAIN_SEPARATOR = computeDomainSeparator();
  57:   }




```


### Recommended Mitigation Steps

Add a control that makes `initialize()` only call the Deployer Contract or EOA;

```js
if (msg.sender != DEPLOYER_ADDRESS) {
            revert NotDeployer();
        }
```

### [L-04] Insufficient coverage

**Description:**
The test coverage rate of the project is ~85%. Testing all functions is best practice in terms of security criteria.
Due to its capacity, test coverage is expected to be 100%.


```js
| File                                                                                | % Lines           | % Statements       | % Branches       | % Funcs          |
|-------------------------------------------------------------------------------------|-------------------|--------------------|------------------|------------------|
| src/utils/EIP165.sol                                                                | 100.00% (0/0)     | 100.00% (0/0)      | 100.00% (0/0)    | 0.00% (0/1)      |
| src/utils/MultiRewardEscrow.sol                                                     | 100.00% (42/42)   | 88.33% (53/60)     | 55.56% (10/18)   | 100.00% (9/9)    |
| src/utils/MultiRewardStaking.sol                                                    | 100.00% (110/110) | 89.78% (123/137)   | 63.04% (29/46)   | 100.00% (26/26)  |
| src/vault/AdminProxy.sol                                                            | 100.00% (1/1)     | 100.00% (1/1)      | 100.00% (0/0)    | 100.00% (1/1)    |
| src/vault/CloneFactory.sol                                                          | 100.00% (5/5)     | 100.00% (6/6)      | 100.00% (4/4)    | 100.00% (1/1)    |
| src/vault/CloneRegistry.sol                                                         | 66.67% (4/6)      | 66.67% (4/6)       | 100.00% (0/0)    | 33.33% (1/3)     |
| src/vault/DeploymentController.sol                                                  | 100.00% (13/13)   | 93.33% (14/15)     | 50.00% (1/2)     | 100.00% (6/6)    |
| src/vault/PermissionRegistry.sol                                                    | 100.00% (8/8)     | 83.33% (10/12)     | 50.00% (2/4)     | 100.00% (3/3)    |
| src/vault/TemplateRegistry.sol                                                      | 100.00% (18/18)   | 80.00% (16/20)     | 50.00% (4/8)     | 100.00% (6/6)    |
| src/vault/Vault.sol                                                                 | 90.00% (108/120)  | 84.83% (123/145)   | 57.14% (24/42)   | 88.57% (31/35)   |
| src/vault/VaultController.sol                                                       | 98.87% (175/177)  | 88.21% (247/280)   | 52.56% (41/78)   | 100.00% (42/42)  |
| src/vault/VaultRegistry.sol                                                         | 60.00% (6/10)     | 63.64% (7/11)      | 100.00% (2/2)    | 33.33% (2/6)     |
| src/vault/adapter/abstracts/AdapterBase.sol                                         | 95.74% (90/94)    | 91.96% (103/112)   | 70.83% (17/24)   | 76.32% (29/38)   |
| src/vault/adapter/abstracts/WithRewards.sol                                         | 0.00% (0/1)       | 0.00% (0/1)        | 100.00% (0/0)    | 0.00% (0/3)      |
| src/vault/adapter/beefy/BeefyAdapter.sol                                            | 80.85% (38/47)    | 79.31% (46/58)     | 55.00% (11/20)   | 69.23% (9/13)    |
| src/vault/adapter/yearn/YearnAdapter.sol                                            | 92.86% (26/28)    | 86.11% (31/36)     | 50.00% (4/8)     | 92.31% (12/13)   |

```


### [L-05] Missing Event for initialize

**Context:**
```solidity
14 results

src/utils/MultiRewardStaking.sol:
  41:   function initialize(

src/vault/Vault.sol:
  57:     function initialize(

src/vault/adapter/beefy/BeefyAdapter.sol:
  46:     function initialize(

src/vault/adapter/yearn/YearnAdapter.sol:
  34:     function initialize(

src/utils/MultiRewardEscrow.sol:
  30:   constructor(address _owner, address _feeRecipient) Owned(_owner) {

src/vault/AdminProxy.sol:
  16:   constructor(address _owner) Owned(_owner) {}

src/vault/CloneFactory.sol:
  23:   constructor(address _owner) Owned(_owner) {}

src/vault/CloneRegistry.sol:
  22:   constructor(address _owner) Owned(_owner) {}

src/vault/DeploymentController.sol:
  35:   constructor(

src/vault/PermissionRegistry.sol:
  20:   constructor(address _owner) Owned(_owner) {}

src/vault/TemplateRegistry.sol:
  24:   constructor(address _owner) Owned(_owner) {}

src/vault/VaultController.sol:
  45:    * @notice Constructor of this contract.
  53:   constructor(

src/vault/VaultRegistry.sol:
  21:   constructor(address _owner) Owned(_owner) {}


```

**Description:**
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
Issuing event-emit during initialization is a detail that many projects skip

**Recommendation:**
Add Event-Emit

### [L-06] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol



### [L-07] Loss of precision due to rounding in ` amount / uint256(rewardsPerSecond) `

Add scalar so roundings are negligible

```solidity
src/vault/adapter/yearn/YearnAdapter.sol:
 25:     uint256 constant DEGRADATION_COEFFICIENT = 10**18;
122:                 ((lockedFundsRatio * lockedProfit) / DEGRADATION_COEFFICIENT);
```


```solidity
src/vault/adapter/yearn/YearnAdapter.sol:

 25:     uint256 constant DEGRADATION_COEFFICIENT = 10**18;

  114:     function _calculateLockedProfit() internal view returns (uint256) {
  115:         uint256 lockedFundsRatio = (block.timestamp - yVault.lastReport()) *
  116:             yVault.lockedProfitDegradation();
  117: 
  118:         if (lockedFundsRatio < DEGRADATION_COEFFICIENT) {
  119:             uint256 lockedProfit = yVault.lockedProfit();
  120:             return
  121:                 lockedProfit -
  122:                 ((lockedFundsRatio * lockedProfit) / DEGRADATION_COEFFICIENT);
  123:         } else {
  124:             return 0;
  125:         }
  126:     }
```

### [L-08]  Use Fuzzing Test for complicated code bases 


I recommend fuzzing testing in complex code structures, especially PopCorn, where there is an innovative code base and a lot of computation.

**Recommendation:**

Use should fuzzing test like Echidna.

As Alberto Cuesta Canada said:

_Fuzzing is not easy, the tools are rough, and the math is hard, but it is worth it. Fuzzing gives me a level of confidence in my smart contracts that I didn’t have before. Relying just on unit testing anymore and poking around in a testnet seems reckless now._



https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05

### [L-09] Add to Event-Emit for critical function


```solidity

src/vault/AdminProxy.sol:
  18    /// @notice Execute arbitrary management functions.
  19:   function execute(address target, bytes calldata callData)
  20:     external
  21:     onlyOwner
  22:     returns (bool success, bytes memory returndata)
  23:   {
  24:     return target.call(callData);
  25:   }

```

### [L-10] The `Timestamp` variable is limited to work until 2106

limiting the timestamp to fit in a uint32 will cause the call below to start reverting in 2106

```solidity

src/utils/MultiRewardStaking.sol:
  220:     event RewardInfoUpdate(IERC20 rewardToken, uint160 rewardsPerSecond, uint32 rewardsEndTimestamp);
  275:     uint32 rewardsEndTimestamp = rewardsPerSecond == 0
  308:     uint32 rewardsEndTimestamp = _calcRewardsEnd(
  340:     uint32 rewardsEndTimestamp = rewards.rewardsEndTimestamp;
  352:     uint32 rewardsEndTimestamp

```

### [L-11] Signature Malleability of EVM's ecrecover()

**Context:**

```solidity
3 results - 3 files

src/utils/MultiRewardStaking.sol:
  458      unchecked {
  459:       address recoveredAddress = ecrecover(
  460          keccak256(

src/vault/Vault.sol:
  677          unchecked {
  678:             address recoveredAddress = ecrecover(
  679                  keccak256(

src/vault/adapter/abstracts/AdapterBase.sol:
  645          unchecked {
  646:             address recoveredAddress = ecrecover(
  647                  keccak256(

```
**Description:**
Description: The function calls the Solidity ecrecover() function directly to verify the given signatures. However, the ecrecover() EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

Although a replay attack seems not possible for this contract, I recommend using the battle-tested OpenZeppelin's ECDSA library.

*Recommendation:**
Use the ecrecover function from OpenZeppelin's ECDSA library for signature verification. (Ensure using a version > 4.7.3 for there was a critical bug >= 4.1.0 < 4.7.3).

### [L-12] Update codes to avoid Compile Errors


```solidity

Compiler run successful (with warnings)
warning[2519]: Warning: This declaration shadows an existing declaration.
   --> src/vault/adapter/abstracts/AdapterBase.sol:388:9:
    |
388 |         uint256 _totalAssets = totalAssets();
    |         ^^^^^^^^^^^^^^^^^^^^
Note: The shadowed declaration is here:
   --> src/vault/adapter/abstracts/AdapterBase.sol:258:5:
    |
258 |     function _totalAssets() internal view virtual returns (uint256) {}
    |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^



warning[2519]: Warning: This declaration shadows an existing declaration.
   --> test/vault/integration/abstract/AbstractVaultIntegrationTest.sol:159:12:
    |
159 |       for (uint256 i; i < 3; ++i) {
    |            ^^^^^^^^^
Note: The shadowed declaration is here:
   --> test/vault/integration/abstract/AbstractVaultIntegrationTest.sol:155:10:
    |
155 |     for (uint256 i; i < len; i++) {
    |          ^^^^^^^^^



warning[2519]: Warning: This declaration shadows an existing declaration.
   --> test/vault/integration/abstract/AbstractVaultIntegrationTest.sol:166:12:
    |
166 |       for (uint256 i; i < 2; ++i) {
    |            ^^^^^^^^^
Note: The shadowed declaration is here:
   --> test/vault/integration/abstract/AbstractVaultIntegrationTest.sol:155:10:
    |
155 |     for (uint256 i; i < len; i++) {
    |          ^^^^^^^^^



warning[2519]: Warning: This declaration shadows an existing declaration.
   --> test/vault/integration/abstract/AbstractVaultIntegrationTest.sol:214:12:
    |
214 |       for (uint256 i; i < 3; ++i) {
    |            ^^^^^^^^^
Note: The shadowed declaration is here:
   --> test/vault/integration/abstract/AbstractVaultIntegrationTest.sol:210:10:
    |
210 |     for (uint256 i; i < len; i++) {
    |          ^^^^^^^^^



warning[2519]: Warning: This declaration shadows an existing declaration.
   --> test/vault/integration/abstract/AbstractVaultIntegrationTest.sol:221:12:
    |
221 |       for (uint256 i; i < 2; ++i) {
    |            ^^^^^^^^^
Note: The shadowed declaration is here:
   --> test/vault/integration/abstract/AbstractVaultIntegrationTest.sol:210:10:
    |
210 |     for (uint256 i; i < len; i++) {
    |          ^^^^^^^^^



warning[9302]: Warning: Return value of low-level calls not used.
   --> src/vault/adapter/abstracts/AdapterBase.sol:444:13:
    |
444 |             address(strategy).delegatecall(
    |             ^ (Relevant source part starts here and spans across multiple lines).



warning[2072]: Warning: Unused local variable.
   --> src/vault/Vault.sol:448:9:
    |
448 |         uint256 highWaterMark_ = highWaterMark;
    |         ^^^^^^^^^^^^^^^^^^^^^^



warning[2072]: Warning: Unused local variable.
   --> test/vault/integration/abstract/AbstractAdapterTest.sol:123:5:
    |
123 |     uint256 callTime = block.timestamp;
    |     ^^^^^^^^^^^^^^^^



warning[2072]: Warning: Unused local variable.
   --> test/vault/integration/abstract/AbstractAdapterTest.sol:475:5:
    |
475 |     uint256 hwm = 1e18;
    |     ^^^^^^^^^^^



warning[2072]: Warning: Unused local variable.
   --> test/vault/integration/abstract/AbstractAdapterTest.sol:481:5:
    |
481 |     uint256 oldTotalAssets = adapter.totalAssets();
    |     ^^^^^^^^^^^^^^^^^^^^^^

```

### [L-13] `_withdraw` event is missing parameters


The `MultiRewardStaking.sol` contract has very important function; `_withdraw` 


However, only amounts are published in emits, whereas msg.sender must be specified in every transaction, a contract or web page listening to events cannot react to users, emit does not serve the purpose.
Basically, this event cannot be used


```diff
src/utils/MultiRewardStaking.sol:
  120    /// @notice Internal withdraw function used by `withdraw()` and `redeem()`. Accrues rewards for the `caller` and `receiver`.
  121:   function _withdraw(
  122:     address caller,
  123:     address receiver,
  124:     address owner,
  125:     uint256 assets,
  126:     uint256 shares
  127:   ) internal override accrueRewards(caller, receiver) {
  128:     if (caller != owner) _approve(owner, msg.sender, allowance(owner, msg.sender) - shares);
  129: 
  130:     _burn(owner, shares);
  131:     IERC20(asset()).safeTransfer(receiver, assets);
  132: 
- 133:     emit Withdraw(caller, receiver, owner, assets, shares);
+ 133:     emit Withdraw(msg.sender, caller, receiver, owner, assets, shares);
  134:   }

```



### Recommended Mitigation Steps
add `msg.sender` parameter in event-emit


### [L-14] Critical Address Changes Should Use Two-step Procedure

The critical procedures should be two step process.


```solidity
src/vault/Vault.sol:
  552       */
  553:     function setFeeRecipient(address _feeRecipient) external onlyOwner {
  554:         if (_feeRecipient == address(0)) revert InvalidFeeRecipient();
  555: 
  556:         emit FeeRecipientUpdated(feeRecipient, _feeRecipient);
  557: 
  558:         feeRecipient = _feeRecipient;
  559:     }

```

Recommended Mitigation Steps
Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.



### [L-15] Lack of Input Validation

For defence-in-depth purpose, it is recommended to perform additional validation against the amount that the user is attempting to deposit, mint, withdraw and redeem to ensure that the submitted amount is valid.

[OpenZeppelinTokenizedVault.sol#L9](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/7c75b8aa89073376fb67d78a40f6d69331092c94/contracts/token/ERC20/extensions/ERC20TokenizedVault.sol#L95)


```diff
src/vault/Vault.sol:
169 */
170: function mint(uint256 shares, address receiver)
171: public
172: nonReentrant
173: whenNotPaused
174: syncFeeCheckpoint
175: returns (uint256 assets)
176: {
177: if (receiver == address(0)) revert InvalidReceiver();
+       require(shares <= maxMint(receiver), "mint more than max");
178: 
179: uint256 depositFee = uint256(fees.deposit);
180: 
181: uint256 feeShares = shares.mulDiv(
182: depositFee,
183: 1e18 - depositFee,
184: Math.Rounding.Down
185: );
186: 
187: assets = convertToAssets(shares + feeShares);
188: 
189: if (feeShares > 0) _mint(feeRecipient, feeShares);
190: 
191: _mint(receiver, shares);
192: 
193: asset.safeTransferFrom(msg.sender, address(this), assets);
194: 
195: adapter.deposit(assets, address(this));
196: 
197: emit Deposit(msg.sender, receiver, assets, shares);
198: }


```

### [L-16] Prevent division by 0

On several locations in the code precautions are not being taken for not dividing by 0, this will revert the code.
These functions can be calledd with 0 value in the input, this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero.

`rewardsPerSecond` can be assign to `0`

```solidity
src/utils/MultiRewardStaking.sol:
  350  
  351:   function _calcRewardsEnd(
  352:     uint32 rewardsEndTimestamp,
  353:     uint160 rewardsPerSecond,
  354:     uint256 amount
  355:   ) internal returns (uint32) {
  356:     if (rewardsEndTimestamp > block.timestamp)
  357:       amount += uint256(rewardsPerSecond) * (rewardsEndTimestamp - block.timestamp);
  358: 
  359:     return (block.timestamp + (amount / uint256(rewardsPerSecond))).safeCastTo32();
  360:   }


```


### [N-01] Omissions in Events

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible:

```diff
src/utils/MultiRewardEscrow.sol:
  207:   function setFees(IERC20[] memory tokens, uint256[] memory tokenFees) external onlyOwner {
  208:     if (tokens.length != tokenFees.length) revert ArraysNotMatching(tokens.length, tokenFees.length);
  209: 
  210:     for (uint256 i = 0; i < tokens.length; i++) {
  211:       if (tokenFees[i] >= 1e17) revert DontGetGreedy(tokenFees[i]);
  212: 
  213:       fees[tokens[i]].feePerc = tokenFees[i];
- 214:       emit FeeSet(tokens[i], tokenFees[i]);
+ 214:       emit FeeSet(tokens[i], tokenFeesold[i], tokenFeesNew[i]); 
  215:     }
  216:   }

```


### [N-02] NatSpec comments should be increased in contracts

**Context:**
All project have 226 functions (without interfaces) but they have only 95 NatSpec’s

**Description:**
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

**Recommendation:**
NatSpec comments should be increased in contracts


### [N-03] You should leave a space after the word "Popcorn Yearn" when concatenating with `string.concat`


```diff

src/vault/adapter/yearn/YearnAdapter.sol:
  34:     function initialize(
  35:         bytes memory adapterInitData,
  36:         address externalRegistry,
  37:         bytes memory
  38:     ) external initializer {
  39:         (address _asset, , , , , ) = abi.decode(
  40:             adapterInitData,
  41:             (address, address, address, uint256, bytes4[8], bytes)
  42:         );
  43:         __AdapterBase_init(adapterInitData);
  44: 
  45:         yVault = VaultAPI(IYearnRegistry(externalRegistry).latestVault(_asset));
  46: 
  47:         _name = string.concat(
-  48:             "Popcorn Yearn",
+ 48:             "Popcorn Yearn ",
  49:             IERC20Metadata(asset()).name(),
  50:             " Adapter"
  51:         );
  52:         _symbol = string.concat("popY-", IERC20Metadata(asset()).symbol());
  53: 
  54:         IERC20(_asset).approve(address(yVault), type(uint256).max);
  55:     }
  56: 
  57      function name()

```

### [N-04] `Function writing` that does not comply with the `Solidity Style Guide`

**Context:**
All Contracts

**Description:**
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

 constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last


### [N-05] Include return parameters in NatSpec comments

**Context:**
All project have 118 returns variable (without interfaces) but they have only 20 @return NatSpec’s


**Description:**
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

**Recommendation:**
Include return parameters in NatSpec comments

_Recommendation  Code Style: (from Uniswap3)_
```js
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

### [N-06] Tokens accidentally sent to the contract cannot be recovered

Using balanceOf for reward distribution means accidentally sent reward tokens to the contract will be distributed to all users who are eligible for rewards. 

Examine and consider the trade-offs of using internal accounting instead of balanceOf(address(this) . Implement functions that allow to withdraw the accidentally sent tokens from the contracts.


**Contex**
MultiRewardStaking.sol


It can't be recovered if the tokens accidentally arrive at the contract address, which has happened to many popular projects, so I recommend adding a recovery code to your critical contracts.

### Recommended Mitigation Steps

Add this code:

```solidity
 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}

```


### [N-07] Take advantage of Custom Error's return value property

An important feature of Custom Error is that values such as address, tokenID, msg.value can 
be written inside the `()` sign, this kind of approach provides a serious advantage in 
debugging and examining the revert details of dapps such as tenderly.


```solidity

22 results - 7 files

src/utils/MultiRewardEscrow.sol:
  95:     if (token == IERC20(address(0))) revert ZeroAddress();
  96:     if (account == address(0)) revert ZeroAddress();
  97:     if (amount == 0) revert ZeroAmount();
  98:     if (duration == 0) revert ZeroAmount();

src/vault/CloneFactory.sol:
  45:     if (!success) revert DeploymentInitFailed();

src/vault/PermissionRegistry.sol:
  40:     if (len != newPermissions.length) revert Mismatch();
  43:       if (newPermissions[i].endorsed && newPermissions[i].rejected) revert Mismatch();

src/vault/Vault.sol:
   74:         if (address(asset_) == address(0)) revert InvalidAsset();
   75:         if (address(asset_) != adapter_.asset()) revert InvalidAdapter();
   90:         if (feeRecipient_ == address(0)) revert InvalidFeeRecipient();
  141:         if (receiver == address(0)) revert InvalidReceiver();
  177:         if (receiver == address(0)) revert InvalidReceiver();
  216:         if (receiver == address(0)) revert InvalidReceiver();
  258:         if (receiver == address(0)) revert InvalidReceiver();
  531:         ) revert InvalidVaultFees();
  554:         if (_feeRecipient == address(0)) revert InvalidFeeRecipient();
  580:             revert VaultAssetMismatchNewAdapterAsset();
  631:             revert InvalidQuitPeriod();

src/vault/VaultController.sol:
  695:       if (!cloneRegistry.cloneExists(adapter)) revert AdapterConfigFaulty();
  701:     if (length1 != length2) revert ArrayLengthMissmatch();

src/vault/VaultRegistry.sol:
  45:     if (metadata[_metadata.vault].vault != address(0)) revert VaultAlreadyRegistered();

src/vault/adapter/beefy/BeefyAdapter.sol:
  222:         if (address(beefyBooster) == address(0)) revert NoBeefyBooster();

```


### [N-08] Repeated validation logic can be converted into a function modifier

If a query or logic is repeated over many lines, using a modifier improves the readability and reusability of the code


```solidity

5 results

src/utils/MultiRewardStaking.sol:
  142:     if (from == address(0) || to == address(0)) revert ZeroAddressTransfer(from, to);
  481:     if (recoveredAddress == address(0) || recoveredAddress != owner) revert InvalidSigner(recoveredAddress);

src/vault/Vault.sol:
  702:     if (recoveredAddress == address(0) || recoveredAddress != owner)

src/vault/VaultController.sol:
  837:     if (address(_deploymentController) == address(0) || address(deploymentController) == address(_deploymentController))

src/vault/adapter/abstracts/AdapterBase.sol:
  670:     if (recoveredAddress == address(0) || recoveredAddress != owner)

```



### [N-09] Use a more recent version of Solidity

**Context:**
```solidity
35 results - 35 files

src/interfaces/IEIP165.sol:
  1  // SPDX-License-Identifier: AGPL-3.0-only
  2: pragma solidity ^0.8.15;
  3  

src/interfaces/IMultiRewardEscrow.sol:
  2  
  3: pragma solidity ^0.8.15;
  4  

src/interfaces/IMultiRewardStaking.sol:
  2  
  3: pragma solidity ^0.8.15;
  4  

src/interfaces/vault/IAdapter.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/interfaces/vault/IAdminProxy.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/interfaces/vault/ICloneFactory.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/interfaces/vault/ICloneRegistry.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/interfaces/vault/IDeploymentController.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/interfaces/vault/IERC4626.sol:
  2  
  3: pragma solidity ^0.8.15;
  4  

src/interfaces/vault/IPermissionRegistry.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/interfaces/vault/IStrategy.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/interfaces/vault/ITemplateRegistry.sol:
  2  
  3: pragma solidity ^0.8.15;
  4  

src/interfaces/vault/IVault.sol:
  2  
  3: pragma solidity ^0.8.15;
  4  

src/interfaces/vault/IVaultController.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/interfaces/vault/IVaultRegistry.sol:
  2  
  3: pragma solidity ^0.8.15;
  4  

src/interfaces/vault/IWithRewards.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/utils/EIP165.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/utils/MultiRewardEscrow.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/utils/MultiRewardStaking.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/AdminProxy.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/CloneFactory.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/CloneRegistry.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/DeploymentController.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/PermissionRegistry.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/TemplateRegistry.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/Vault.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/VaultController.sol:
  2  
  3: pragma solidity ^0.8.15;
  4  

src/vault/VaultRegistry.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/adapter/abstracts/AdapterBase.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/adapter/abstracts/OnlyStrategy.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/adapter/abstracts/WithRewards.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/adapter/beefy/BeefyAdapter.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/adapter/beefy/IBeefy.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/adapter/yearn/IYearn.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/adapter/yearn/YearnAdapter.sol:
  3  
  4: pragma solidity ^0.8.15;

```
**Description:**
For security, it is best practice to use the latest Solidity version.
For the security fix list in the versions;
https://github.com/ethereum/solidity/blob/develop/Changelog.md


**Recommendation:**
Old version of Solidity is used `(0.8.15)`, newer version can be used `(0.8.17)` 




### [N-10] For functions, follow Solidity standard naming conventions (internal function style rule)

**Context:**
```solidity

4 results - 4 files

src/utils/MultiRewardEscrow.sol:
  71:   uint256 internal nonce;

src/utils/MultiRewardStaking.sol:
  491:   function computeDomainSeparator() internal view virtual returns (bytes32) {

src/vault/Vault.sol:
  716:     function computeDomainSeparator() internal view virtual returns (bytes32) {

src/vault/adapter/abstracts/AdapterBase.sol:
  684:     function computeDomainSeparator() internal view virtual returns (bytes32) {

```


**Description:**
The above codes don't follow Solidity's standard naming convention,

internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

### [N-11] Floating pragma

**Description:**
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
https://swcregistry.io/docs/SWC-103

Floating Pragma List: 
```solidity
35 results - 35 files

src/interfaces/IEIP165.sol:
  2: pragma solidity ^0.8.15;

src/interfaces/IMultiRewardEscrow.sol:
  3: pragma solidity ^0.8.15;
  4  

src/interfaces/IMultiRewardStaking.sol:
  2  
  3: pragma solidity ^0.8.15;
  4  

src/interfaces/vault/IAdapter.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/interfaces/vault/IAdminProxy.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/interfaces/vault/ICloneFactory.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/interfaces/vault/ICloneRegistry.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/interfaces/vault/IDeploymentController.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/interfaces/vault/IERC4626.sol:
  2  
  3: pragma solidity ^0.8.15;
  4  

src/interfaces/vault/IPermissionRegistry.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/interfaces/vault/IStrategy.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/interfaces/vault/ITemplateRegistry.sol:
  2  
  3: pragma solidity ^0.8.15;
  4  

src/interfaces/vault/IVault.sol:
  2  
  3: pragma solidity ^0.8.15;
  4  

src/interfaces/vault/IVaultController.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/interfaces/vault/IVaultRegistry.sol:
  2  
  3: pragma solidity ^0.8.15;
  4  

src/interfaces/vault/IWithRewards.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/utils/EIP165.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/utils/MultiRewardEscrow.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/utils/MultiRewardStaking.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/AdminProxy.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/CloneFactory.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/CloneRegistry.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/DeploymentController.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/PermissionRegistry.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/TemplateRegistry.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/Vault.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/VaultController.sol:
  2  
  3: pragma solidity ^0.8.15;
  4  

src/vault/VaultRegistry.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/adapter/abstracts/AdapterBase.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/adapter/abstracts/OnlyStrategy.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/adapter/abstracts/WithRewards.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/adapter/beefy/BeefyAdapter.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/adapter/beefy/IBeefy.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/adapter/yearn/IYearn.sol:
  3  
  4: pragma solidity ^0.8.15;
  5  

src/vault/adapter/yearn/YearnAdapter.sol:
  3  
  4: pragma solidity ^0.8.15;

```


**Recommendation:**
Lock the pragma version and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.


### [N-12] Use descriptive names for Contracts and Libraries


This codebase will be difficult to navigate, as there are no descriptive naming conventions that specify which files should contain meaningful logic.

Prefixes should be added like this by filing:

Interface I_
absctract contracts Abs_
Libraries Lib_

We recommend that you implement this or a similar agreement.



### [N-13] Use SMTChecker

The *highest* tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19



### [N-14]  Add NatSpec Mapping comment

**Description:**
Add NatSpec comments describing mapping keys and values


**Context:**

```solidity
8 results - 6 files

src/vault/CloneRegistry.sol:
  28:   mapping(address => bool) public cloneExists;

src/vault/PermissionRegistry.sol:
  26:   mapping(address => Permission) public permissions;

src/vault/TemplateRegistry.sol:
  32:   mapping(bytes32 => bytes32[]) public templateIds;
  33:   mapping(bytes32 => bool) public templateExists;
  35:   mapping(bytes32 => bool) public templateCategoryExists;

src/vault/Vault.sol:
  659:     mapping(address => uint256) public nonces;

src/vault/VaultController.sol:
  851:   mapping(bytes32 => bytes32) public activeTemplateId;

src/vault/adapter/abstracts/AdapterBase.sol:
  627:     mapping(address => uint256) public nonces;

```

**Recommendation:**

```solidity

 /// @dev    address(user) -> address(ERC0 Contract Address) -> uint256(allowance amount from user)
    mapping(address => mapping(address => uint256)) public allowance;

```

### [N-15] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier


```diff
6 results - 3 files

src/vault/Vault.sol:
-  35:     uint256 constant SECONDS_PER_YEAR = 365.25 days;
+ 35:     uint256 constant SECONDS_PER_YEAR = 365.25 days; // 31_536_000  31_557_600  (365.25*24*60*60)

-  619:     uint256 public quitPeriod = 3 days;
+  619:     uint256 public quitPeriod = 3 days; //  259_200 (3*24*60*60)

-  630:     if (_quitPeriod < 1 days || _quitPeriod > 7 days)
+  630:     if (_quitPeriod < 1 days || _quitPeriod > 7 days) //  604_800 (7*24*60*60)


src/vault/VaultController.sol:
- 793:     if (newCooldown > 1 days) revert InvalidHarvestCooldown(newCooldown);
+ 793:     if (newCooldown > 1 days) revert InvalidHarvestCooldown(newCooldown); // 86_400  (1*24*60*60)


src/vault/adapter/abstracts/AdapterBase.sol:
- 502:         if (newCooldown >= 1 days) revert InvalidHarvestCooldown(newCooldown);
+ 502:         if (newCooldown >= 1 days) revert InvalidHarvestCooldown(newCooldown); // 86_400  (1*24*60*60)

```

### [N-16] Lines are too long

[AdapterBase.sol#L6](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L6)

[AdapterBase.sol#L52](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L52)

[VaultController.sol#L477](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L477)

[MultiRewardStaking.sol#L7](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L7)

[MultiRewardStaking.sol#L380](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L380)

[BeefyAdapter.sol#L16](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L16)

[MultiRewardEscrow.sol#L49](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L49)


**Description:**
It is generally recommended that lines in the source code should not exceed 80-120 characters. Today's screens are much larger, so in some cases it makes sense to expand that.The lines above should be split when they reach that length, as the files will most likely be on GitHub and GitHub always uses a scrollbar when the length is more than 164 characters.

(why-is-80-characters-the-standard-limit-for-code-width)[https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width]

**Recommendation:**
Multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the Maximum Line Length section.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html#introduction

```js
thisFunctionCallIsReallyLong(
    longArgument1,
    longArgument2,
    longArgument3
);
```



### [N-17] Open TODOs

**Context:**
```solidity

src/vault/adapter/abstracts/AdapterBase.sol:
  515  
  516:     // TODO use deterministic fee recipient proxy
  517:     address FEE_RECIPIENT = address(0x4444);


```

**Recommendation:**
Use temporary TODOs as you work on a feature, but make sure to treat them before merging. Either add a link to a proper issue in your TODO, or remove it from the code.



### [N-18] Constants on the left are better

If you use the constant first you support structures that veil programming errors. And one should only produce code either to add functionality, fix an programming error or trying to support structures to avoid programming errors (like design patterns). 

https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html

```solidity

9 results - 3 files

src/utils/MultiRewardEscrow.sol:
   97:     if (amount == 0) revert ZeroAmount();
   98:     if (duration == 0) revert ZeroAmount();
  160:       if (claimable == 0) revert NotClaimable(escrowId);

src/utils/MultiRewardStaking.sol:
  174:       if (rewardAmount == 0) revert ZeroRewards(_rewardTokens[i]);
  258:       if (rewardsPerSecond == 0) revert ZeroRewardsSpeed();
  299:     if (rewardsPerSecond == 0) revert ZeroAmount();
  325:     if (amount == 0) revert ZeroAmount();
  420:     if (oldIndex == 0) {

src/vault/adapter/yearn/YearnAdapter.sol:
  90:         if (yVault.totalSupply() == 0) return yShares;


```

### [N-19] Confusing event-emit argument names

In the naming of event-emit arguments; Naming can be confusing when using pre and current value

```diff

src/vault/Vault.sol:
- 553:     function setFeeRecipient(address _feeRecipient) external onlyOwner {
+ 553:     function setFeeRecipient(address NewfeeRecipient) external onlyOwner {

- 554:         if (_feeRecipient == address(0)) revert InvalidFeeRecipient();
+ 554:         if (NewfeeRecipient == address(0)) revert InvalidFeeRecipient();

  555: 
- 556:         emit FeeRecipientUpdated(feeRecipient, _feeRecipient);
+ 556:        emit FeeRecipientUpdated(oldfeeRecipient, NewfeeRecipient);

  557: 
- 558:         feeRecipient = _feeRecipient;
+ 558:        feeRecipient = NewfeeRecipient;

  559:     }

```


### [N-20] `constants` should be defined rather than using magic numbers

Even assembly can benefit from using readable constants instead of hex/numeric literals

```solidity


src/vault/Vault.sol:
   34  
   35:     uint256 constant SECONDS_PER_YEAR = 365.25 days;
  619:     uint256 public quitPeriod = 3 days;
```

### [N-21] Missing timelock for critical parameter change

Changing `fee` makes a critical and important value change for users, therefore it is recommended to use timelock

```solidity

1 result - 1 file

src/vault/VaultController.sol:
  750     */
  751:   function setPerformanceFee(uint256 newFee) external onlyOwner {
  752:     // Dont take more than 20% performanceFee
  753:     if (newFee > 2e17) revert InvalidPerformanceFee(newFee);
  754: 
  755:     emit PerformanceFeeChanged(performanceFee, newFee);
  756: 
  757:     performanceFee = newFee;
  758:   }

```


### [N-22] Use the `delete` keyword instead of assigning a value of `0`


Using the 'delete' keyword instead of assigning a '0' value is a detailed optimization that increases code readability and audit quality, and clearly indicates the intent.

Other hand, if use `delete` instead `0` value assign , it will be gas saved

```diff

2 results - 2 files

src/utils/MultiRewardStaking.sol:
- 186:       accruedRewards[user][_rewardTokens[i]] = 0;
+ 186:      delete  accruedRewards[user][_rewardTokens[i]];

src/vault/adapter/abstracts/AdapterBase.sol:
- 577:         underlyingBalance = 0;
+ 577:         delete underlyingBalance;
```

### [N-23] Variable names that consist of all capital letters should be reserved for only `constant/immutable` variables


```solidity


6 results - 3 files

src/utils/MultiRewardStaking.sol:
  438:   uint256 internal INITIAL_CHAIN_ID;
  439:   bytes32 internal INITIAL_DOMAIN_SEPARATOR;

src/vault/Vault.sol:
  657:     uint256 internal INITIAL_CHAIN_ID;
  658:     bytes32 internal INITIAL_DOMAIN_SEPARATOR;

src/vault/adapter/abstracts/AdapterBase.sol:
  625:     uint256 internal INITIAL_CHAIN_ID;
  626:     bytes32 internal INITIAL_DOMAIN_SEPARATOR;

```


## [N-24] Avoid _shadowing_ `inherited state variables`

**Context:**
[MultiRewardStaking.sol#L121-L134](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L121-L134)

**Description:**
In ``` MultiRewardStaking.sol#L121-L134 ``` , there is a local variable named ```owner```, but there is a state varible named ```owner``` in the inherited ```OwnedUpgradeable.sol``` with the same name. This use causes compilers to issue warnings, negatively affecting checking and code readability.

And some IDE and code check programs give this warning:

```js
INHERITED ❗SHADOWED❗(null) StateVar OwnedUpgradeable.owner
```

```js
src/utils/MultiRewardStaking.sol:

  121:   function _withdraw(
  122:     address caller,
  123:     address receiver,
  124:     address owner,
  125:     uint256 assets,
  126:     uint256 shares
  127:   ) internal override accrueRewards(caller, receiver) {
  128:     if (caller != owner) _approve(owner, msg.sender, allowance(owner, msg.sender) - shares);
  129: 
  130:     _burn(owner, shares);
  131:     IERC20(asset()).safeTransfer(receiver, assets);
  132: 
  133:     emit Withdraw(caller, receiver, owner, assets, shares);
  134:   }


```

**Recommendation:**
Avoid using variables with the same name, including inherited in the same contract, if used, it must be specified in the NatSpec comments.

### [N-25] Not using the named return variables anywhere in the function is confusing


```solidity
src/vault/AdminProxy.sol:
  18    /// @notice Execute arbitrary management functions.
  19:   function execute(address target, bytes calldata callData)
  20:     external
  21:     onlyOwner
  22:     returns (bool success, bytes memory returndata)
  23:   {
  24:     return target.call(callData);
  25:   }
  26  }

```

**Recommendation:**
Consider adopting a consistent approach to return values throughout the codebase by removing all named return variables, explicitly declaring them as local variables, and adding the necessary return statements where appropriate. This would improve both the explicitness and readability of the code, and it may also help reduce regressions during future code refactors.

### [N-26] Unnecessary Emission of block.timestamp



```solidity

2 result 

src/vault/Vault.sol:
  525:     function proposeFees(VaultFees calldata newFees) external onlyOwner {
  526:         if (
  527:             newFees.deposit >= 1e18 ||
  528:             newFees.withdrawal >= 1e18 ||
  529:             newFees.management >= 1e18 ||
  530:             newFees.performance >= 1e18
  531:         ) revert InvalidVaultFees();
  532: 
  533:         proposedFees = newFees;
  534:         proposedFeeTime = block.timestamp;
  535: 
  536:         emit NewFeesProposed(newFees, block.timestamp);
  537:     }


src/vault/Vault.sol:
  578:     function proposeAdapter(IERC4626 newAdapter) external onlyOwner {
  579:         if (newAdapter.asset() != address(asset))
  580:             revert VaultAssetMismatchNewAdapterAsset();
  581: 
  582:         proposedAdapter = newAdapter;
  583:         proposedAdapterTime = block.timestamp;
  584: 
  585:         emit NewAdapterProposed(newAdapter, block.timestamp);
  586:     }

```

**Recommendation:**
Any event emitted by Solidity includes the timestamp and block number implicitly. Adding it the second time just increases the cost of the transaction

### [N-27] Function Calls in Loop Could Lead to Denial of Service due to Array length not being checked


```diff
src/utils/MultiRewardEscrow.sol:
  153     */
  154:   function claimRewards(bytes32[] memory escrowIds) external {
+              require(escrowIds.length< max escrowIdsLengt, "max length");

  155:     for (uint256 i = 0; i < escrowIds.length; i++) {
  156:       bytes32 escrowId = escrowIds[i];
  157:       Escrow memory escrow = escrows[escrowId];
  158: 
  159:       uint256 claimable = _getClaimableAmount(escrow);
  160:       if (claimable == 0) revert NotClaimable(escrowId);
  161: 
  162:       escrows[escrowId].balance -= claimable;
  163:       escrows[escrowId].lastUpdateTime = block.timestamp.safeCastTo32();
  164: 
  165:       escrow.token.safeTransfer(escrow.account, claimable);
  166:       emit RewardsClaimed(escrow.token, escrow.account, claimable);
  167:     }
  168:   }


```

**Recommendation:**
Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to gas limitations or failed transactions. Consider bounding the loop length if the array is expected to be growing and/or handling a huge list of elements to avoid unnecessary gas wastage and denial of service.

### [N-28] Don't use _msgSender() if not supporting EIP-2771

Use ``msg.sender`` if the code does not implement EIP-2771 trusted forwarder support.

Reference: https://eips.ethereum.org/EIPS/eip-2771


```solidity
4 results - 1 file

src/vault/adapter/abstracts/AdapterBase.sol:
  118          uint256 shares = _previewDeposit(assets);
  119:         _deposit(_msgSender(), receiver, assets, shares);
  120  

  137          uint256 assets = _previewMint(shares);
  138:         _deposit(_msgSender(), receiver, assets, shares);
  139  

  181  
  182:         _withdraw(_msgSender(), receiver, owner, assets, shares);
  183  

  200          uint256 assets = _previewRedeem(shares);
  201:         _withdraw(_msgSender(), receiver, owner, assets, shares);


```
