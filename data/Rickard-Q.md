# [L-01] Upon module's upgrade, the token approval sould be revoked 

## Context
[VaultController.sol#L456](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L456), [BeefyAdapter.sol#L80](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L80), [BeefyAdapter.sol#L83](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L83), [YearnAdapter.sol#L54](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L54)
## Description
There is no logic to revoke these approvals (e.i. approve to zero). In the case of the `address()` has some bugs, it may be desirable to be able to revoke the approvals.   
[VaultController.sol#L456](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L456)
````solidity
            IERC20(rewardsToken).approve(staking, type(uint256).max);
````
[BeefyAdapter.sol#L80](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L80), [BeefyAdapter.sol#L83](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L83)
````solidity
            IERC20(asset()).approve(_beefyVault, type(uint256).max);
            IERC20(_beefyVault).approve(_beefyBooster, type(uint256).max);

````
[YearnAdapter.sol#L54](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L54)
````solidity
            IERC20(_asset).approve(address(yVault), type(uint256).max);
````
## Recommendations
Consider implementing an approval to `0`.


# [L-02] Usage of deprecated `safeApprove()`

## Context
[MultiRewardStaking.sol#L271](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L271)
## Description
OpenZeppelin `safeApprove()` implementation is deprecated. Reference. Using this deprecated function can lead to unintended reverts and potential locking of funds.   
````solidity
      rewardToken.safeApprove(address(escrow), type(uint256).max);
````
## Recommendations
Consider replacing `safeApprove()` with `safeIncreaseAllowance()` or `safeDecreaseAllowance()` instead regarding Openzeppelin comment.
````solidity
-      rewardToken.safeApprove(address(escrow), type(uint256).max);
+      rewardToken.safeDecreaseAllowance(address(escrow), type(uint256).max);
````

# [I-01]  Use of floating `pragma` version

## Context
Present in most files.
## Description
 Contracts should be deployed using a fixed `pragma` version. Locking the pragma helps to ensure that
contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce
bugs that affect the contract system negatively   

## Recommendations
Lock the pragma version and also consider upgrading the project pragma version to a newer
stable version, currently 0.8.10.


# [I-02]  Missing natspec comments for contract's constructor, variables or functions

## Context
Present in most files.
## Description
Some of the contract's `constructor` variables and functions are missing natespec comments.

## Recommendations
Consider adding the missing natspec comments.


# [I-03] Use `bytes.concat` instead of `abi.encodePacked` for concatenation

## Context
[MultiRewardEscrow.sol#L104](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L104), [MultiRewardStaking.sol#L49-L50](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L49-L50), [MultiRewardStaking.sol#L459-L479](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L459-L479)
## Description
 While one of the uses of `abi.encodePacked` is to perform concatenation, the Solidity language does
contain a reserved function for this: `bytes.concat`.
## Recommendations
 Consider using bytes.concat instead of abi.encodePacked for concatenation:   
 [MultiRewardEscrow.sol#L104](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L104)
````solidity
-            bytes32 id = keccak256(abi.encodePacked(token, account, amount, nonce));
+            bytes32 id = keccak256(bytes.concat(token, account, amount, nonce));
````
[MultiRewardStaking.sol#L49-L50](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L49-L50)
````solidity
-            _name = string(abi.encodePacked("Staked ", IERC20Metadata(address(_stakingToken)).name()));
-            _symbol = string(abi.encodePacked("pst-", IERC20Metadata(address(_stakingToken)).symbol()));

+            _name = string(bytes.concat("Staked ", IERC20Metadata(address(_stakingToken)).name()));
+            _symbol = string(bytes.concat("pst-", IERC20Metadata(address(_stakingToken)).symbol()));
````
[MultiRewardStaking.sol#L459-L479](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L459-L479)
````solidity
      address recoveredAddress = ecrecover(
        keccak256(
-         abi.encodePacked(
+         bytes.concat(
            "\x19\x01",
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
````
# [N-01] Insufficient coverage 

## Description
The test coverage rate of the project is not fulfilled. Testing all functions is best practice in terms of security criteria.   
````solidity
| File                                                                                | % Lines           | % Statements       | % Branches       | % Func
s          |
|-------------------------------------------------------------------------------------|-------------------|--------------------|------------------|-------
-----------|
| node_modules/@openzeppelin/contracts/proxy/Clones.sol                               | 0.00% (0/6)       | 0.00% (0/6)        | 0.00% (0/4)      | 0.00%
(0/4)      |
| node_modules/@openzeppelin/contracts/token/ERC20/ERC20.sol                          | 83.64% (46/55)    | 80.65% (50/62)     | 45.45% (10/22)   | 77.78%
 (14/18)   |
| node_modules/@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol                | 0.00% (0/17)      | 0.00% (0/23)       | 0.00% (0/10)     | 0.00%
(0/7)      |
| node_modules/@openzeppelin/contracts/utils/Address.sol                              | 0.00% (0/26)      | 0.00% (0/28)       | 0.00% (0/16)     | 0.00%
(0/13)     |
| node_modules/@openzeppelin/contracts/utils/Context.sol                              | 50.00% (1/2)      | 50.00% (1/2)       | 100.00% (0/0)    | 50.00%
 (1/2)     |
| node_modules/@openzeppelin/contracts/utils/Strings.sol                              | 0.00% (0/18)      | 0.00% (0/25)       | 0.00% (0/4)      | 0.00%
(0/4)      |
| node_modules/@openzeppelin/contracts/utils/math/Math.sol                            | 6.09% (7/115)     | 5.69% (7/123)      | 2.08% (1/48)     | 14.29%
 (2/14)    |
| node_modules/openzeppelin-upgradeable/proxy/utils/Initializable.sol                 | 0.00% (0/4)       | 0.00% (0/4)        | 0.00% (0/4)      | 0.00%
(0/1)      |
| node_modules/openzeppelin-upgradeable/security/PausableUpgradeable.sol              | 100.00% (9/9)     | 100.00% (9/9)      | 75.00% (3/4)     | 100.00
% (7/7)    |
| node_modules/openzeppelin-upgradeable/security/ReentrancyGuardUpgradeable.sol       | 0.00% (0/2)       | 0.00% (0/2)        | 100.00% (0/0)    | 0.00%
(0/2)      |
| node_modules/openzeppelin-upgradeable/token/ERC20/ERC20Upgradeable.sol              | 68.97% (40/58)    | 67.69% (44/65)     | 31.82% (7/22)    | 80.00%
 (16/20)   |
| node_modules/openzeppelin-upgradeable/token/ERC20/extensions/ERC4626Upgradeable.sol | 76.74% (33/43)    | 77.55% (38/49)     | 40.00% (4/10)    | 69.57%
 (16/23)   |
| node_modules/openzeppelin-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol    | 0.00% (0/17)      | 0.00% (0/23)       | 0.00% (0/10)     | 0.00%
(0/7)      |
| node_modules/openzeppelin-upgradeable/utils/AddressUpgradeable.sol                  | 0.00% (0/19)      | 0.00% (0/21)       | 0.00% (0/14)     | 0.00%
(0/9)      |
| node_modules/openzeppelin-upgradeable/utils/ContextUpgradeable.sol                  | 50.00% (1/2)      | 50.00% (1/2)       | 100.00% (0/0)    | 25.00%
 (1/4)     |
| node_modules/openzeppelin-upgradeable/utils/math/MathUpgradeable.sol                | 0.00% (0/69)      | 0.00% (0/73)       | 0.00% (0/24)     | 0.00%
(0/8)      |
| src/utils/EIP165.sol                                                                | 100.00% (0/0)     | 100.00% (0/0)      | 100.00% (0/0)    | 0.00%
(0/1)      |
| src/utils/MultiRewardEscrow.sol                                                     | 100.00% (42/42)   | 88.33% (53/60)     | 55.56% (10/18)   | 100.00
% (9/9)    |
| src/utils/MultiRewardStaking.sol                                                    | 100.00% (110/110) | 89.78% (123/137)   | 63.04% (29/46)   | 100.00
% (26/26)  |
| src/utils/Owned.sol                                                                 | 100.00% (7/7)     | 100.00% (7/7)      | 75.00% (3/4)     | 100.00
% (3/3)    |
| src/utils/OwnedUpgradeable.sol                                                      | 40.00% (4/10)     | 40.00% (4/10)      | 33.33% (2/6)     | 50.00%
 (2/4)     |
| src/vault/AdminProxy.sol                                                            | 100.00% (1/1)     | 100.00% (1/1)      | 100.00% (0/0)    | 100.00
% (1/1)    |
| src/vault/CloneFactory.sol                                                          | 100.00% (5/5)     | 100.00% (6/6)      | 100.00% (4/4)    | 100.00
% (1/1)    |
| src/vault/CloneRegistry.sol                                                         | 66.67% (4/6)      | 66.67% (4/6)       | 100.00% (0/0)    | 33.33%
 (1/3)     |
| src/vault/DeploymentController.sol                                                  | 100.00% (13/13)   | 93.33% (14/15)     | 50.00% (1/2)     | 100.00
% (6/6)    |
| src/vault/PermissionRegistry.sol                                                    | 100.00% (8/8)     | 83.33% (10/12)     | 50.00% (2/4)     | 100.00
% (3/3)    |
| src/vault/TemplateRegistry.sol                                                      | 100.00% (18/18)   | 80.00% (16/20)     | 50.00% (4/8)     | 100.00
% (6/6)    |
| src/vault/Vault.sol                                                                 | 90.00% (108/120)  | 84.83% (123/145)   | 57.14% (24/42)   | 88.57%
 (31/35)   |
| src/vault/VaultController.sol                                                       | 98.87% (175/177)  | 88.21% (247/280)   | 52.56% (41/78)   | 100.00
% (42/42)  |
| src/vault/VaultRegistry.sol                                                         | 60.00% (6/10)     | 63.64% (7/11)      | 100.00% (2/2)    | 33.33%
 (2/6)     |
| src/vault/adapter/abstracts/AdapterBase.sol                                         | 95.74% (90/94)    | 91.96% (103/112)   | 70.83% (17/24)   | 76.32%
 (29/38)   |
| src/vault/adapter/abstracts/WithRewards.sol                                         | 0.00% (0/1)       | 0.00% (0/1)        | 100.00% (0/0)    | 0.00%
(0/3)      |
| src/vault/adapter/beefy/BeefyAdapter.sol                                            | 80.85% (38/47)    | 79.31% (46/58)     | 55.00% (11/20)   | 69.23%
 (9/13)    |
| src/vault/adapter/yearn/YearnAdapter.sol                                            | 92.86% (26/28)    | 86.11% (31/36)     | 50.00% (4/8)     | 92.31%
 (12/13)   |
| src/vault/strategy/Pool2SingleAssetCompounder.sol                                   | 0.00% (0/28)      | 0.00% (0/47)       | 0.00% (0/4)      | 0.00%
(0/3)      |
| src/vault/strategy/RewardsClaimer.sol                                               | 0.00% (0/9)       | 0.00% (0/15)       | 100.00% (0/0)    | 0.00%
(0/1)      |
| src/vault/strategy/StrategyBase.sol                                                 | 0.00% (0/4)       | 0.00% (0/9)        | 0.00% (0/4)      | 0.00%
(0/4)      |
| test/utils/EnhancedTest.sol                                                         | 0.00% (0/38)      | 0.00% (0/42)       | 0.00% (0/16)     | 0.00%
(0/4)      |
| test/utils/Faucet.sol                                                               | 0.00% (0/27)      | 0.00% (0/29)       | 100.00% (0/0)    | 0.00%
(0/8)      |
| test/utils/mocks/ClonableWithInitData.sol                                           | 100.00% (1/1)     | 100.00% (1/1)      | 100.00% (0/0)    | 100.00
% (1/1)    |
| test/utils/mocks/ClonableWithoutInitData.sol                                        | 100.00% (2/2)     | 100.00% (2/2)      | 100.00% (0/0)    | 100.00
% (2/2)    |
| test/utils/mocks/MockAdapter.sol                                                    | 100.00% (11/11)   | 100.00% (11/11)    | 100.00% (2/2)    | 100.00
% (6/6)    |
| test/utils/mocks/MockERC20.sol                                                      | 66.67% (2/3)      | 66.67% (2/3)       | 100.00% (0/0)    | 66.67%
 (2/3)     |
| test/utils/mocks/MockERC4626.sol                                                    | 70.00% (28/40)    | 67.39% (31/46)     | 50.00% (4/8)     | 50.00%
 (9/18)    |
| test/utils/mocks/MockStrategy.sol                                                   | 100.00% (4/4)     | 100.00% (4/4)      | 100.00% (0/0)    | 100.00
% (4/4)    |
| test/vault/integration/abstract/PropertyTest.prop.sol                               | 0.00% (0/86)      | 0.00% (0/126)      | 0.00% (0/8)      | 0.00%
(0/16)     |
| test/vault/integration/beefy/BeefyTestConfigStorage.sol                             | 100.00% (2/2)     | 100.00% (2/2)      | 100.00% (0/0)    | 100.00
% (2/2)    |
| test/vault/integration/beefy/BeefyVault.t.sol                                       | 7.69% (2/26)      | 6.45% (2/31)       | 100.00% (0/0)    | 0.00%
(0/5)      |
| test/vault/integration/yearn/YearnTestConfigStorage.sol                             | 100.00% (2/2)     | 100.00% (2/2)      | 100.00% (0/0)    | 100.00
% (2/2)    |
| test/vault/integration/yearn/YearnVault.t.sol                                       | 7.14% (1/14)      | 6.25% (1/16)       | 100.00% (0/0)    | 0.00%
(0/5)      |
| Total                                                                               | 58.17% (847/1456) | 55.11% (1003/1820) | 37.00% (185/500) | 60.91%
 (268/440) |
````
Test coverage is expected to be 100%.


# [N-02] Functions, parameters and variables in snake case

## Context
[IDeploymentController.sol#L38](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IDeploymentController.sol#L38)
## Description
The specified function does not follow the guideline and grammar adopted throughout the rest of the protocol.
## Recommendations
Use camel case for all functions, parameters and variables and snake case for constants.
````solidity
-            function PermissionRegistry() external view returns (IPermissionRegistry);
+            function permissionRegistry() external view returns (IPermissionRegistry);
````


# [N-03] Use scientific notation (e.g. 1E18) rather than exponentation (e.g. 10**18)

## Context
[MultiRewardStaking.sol#L274](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L274), [YearnAdapter.sol#L25](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L25)
## Description
Scientific notation should be used for better code readability.
## Recommendations
Change the line of code as mentioned below:   
[MultiRewardStaking.sol#L274](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L274)
````solidity
-            uint64 ONE = (10**IERC20Metadata(address(rewardToken)).decimals()).safeCastTo64();
+            uint64 ONE = (10eIERC20Metadata(address(rewardToken)).decimals()).safeCastTo64();
````
[YearnAdapter.sol#L25](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L25)
````solidity
-            uint256 constant DEGRADATION_COEFFICIENT = 10**18;
+            uint256 constant DEGRADATION_COEFFICIENT = 10e18;
````


# [N-04] `Pragma` version ^0.8.15 is too recent to be trusted

## Context
Present in most files.
## Description
[Scientific notation should be used for better code readability.](https://github.com/ethereum/solidity/blob/develop/Changelog.md)   
0.8.17 (2022-09-08)   
0.8.16 (2022-08-08)   
0.8.15 (2022-06-15)   
0.8.10 (2021-11-09)   

Unexpected bugs can be reported in recent versions.  
Risks related to recent releases.  
Risks of complex code generation changes.  
Risks of new language features.  
Risks of known bugs.  
## Recommendations
Use a non-legacy and more battle-tested version.
Use 0.8.10.


# [N-05] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier

## Context
[Vault.sol#L35](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L35), [Vault.sol#L466](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L466), [Vault.sol#L619](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L619)
## Description
Scientific notation should be used for better code readability.
## Recommendations
Change the line of code as mentioned below:   
[MultiRewardStaking.sol#L274](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L274)
````solidity
-            uint256 constant SECONDS_PER_YEAR = 365.25 days;
+            uint256 constant SECONDS_PER_YEAR = 365.25 days; // 31_536_000 ( 365.25 * 24 * 60 * 60) 

-            uint256 public highWaterMark = 1e18;
+            uint256 public highWaterMark = 1e18; // 1_000_000_000_000_000_000

-            uint256 public quitPeriod = 3 days;
+            uint256 public quitPeriod = 3 days; // 259_200 ( 3 * 24 * 60 * 60)
````


# [N-06] Function writing that does not comply with the solidity style guide

## Context
Present in most files.
## Description
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.   

https://docs.soliditylang.org/en/v0.8.17/style-guide.html
## Recommendations
Functions should be grouped according to their visibility and ordered:   
   
- constructor   
- receive function (if exists)    
- fallback function (if exists)    
- external    
- public   
- internal   
- private   
- within a grouping, place the view and pure functions last   

