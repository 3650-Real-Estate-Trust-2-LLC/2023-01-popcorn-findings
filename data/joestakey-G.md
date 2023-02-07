# Gas Report

## Table of Contents
|      | Issue                                                                                             |
|------|---------------------------------------------------------------------------------------------------|
| G-01 | some variables in `MultiRewardEscrow` `DeploymentController` and `VaultController` can be immutable  |


## Summary

> A simple change in variables declarations allows to save significant amounts of gas on function calls. This does make deployments more expensive, but the trade-off is deemed worth it to reduce the gas cost for protocol users
> The total gas saved on users function calls is `47,222`


# [G-01] some variabeles in `MultiRewardEscrow` `DeploymentController` and `VaultController` can be immutable

Variables only set once in the constructor can be made immutable, saving a `SLOAD` operation every time they are called in functions.


## Code changes

```diff
File: src/utils/MultiRewardEscrow.sol
-191:   address public feeRecipient;
+191:   address public immutable feeRecipient;

```

```diff
File: src/vault/DeploymentController.sol
-23:   ICloneFactory public cloneFactory;
-24:   ICloneRegistry public cloneRegistry;
-25:   ITemplateRegistry public templateRegistry;
+23:   ICloneFactory public immutable cloneFactory;
+24:   ICloneRegistry public immutable cloneRegistry;
+25:   ITemplateRegistry public immutable templateRegistry;
```

```diff
File: src/vault/VaultController.sol
-387:   IVaultRegistry public vaultRegistry;

-535:   IMultiRewardEscrow public escrow;

-717:   IAdminProxy public adminProxy;

-822:   IPermissionRegistry public permissionRegistry;

+387:   IVaultRegistry public immutable vaultRegistry;

+535:   IMultiRewardEscrow public immutable escrow;

+717:   IAdminProxy public immutable adminProxy;

+822:   IPermissionRegistry public immutable permissionRegistry;
```


## Tools Used

Manual Analysis, Foundry

## Savings

`MultiRewardEscrow`
- this saves `2,100` gas on `lock()` calls when there is a fee.


`DeploymentController` BEFORE 
|                                        |                 |        |        |        |         |
|----------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                        | Deployment Size |        |        |        |         |
| 852269                                 | 4360            |        |        |        |         |
| Function Name                          | min             | avg    | median | max    | # calls |
| acceptDependencyOwnership              | 7030            | 7030   | 7030   | 7030   | 116     |
| acceptOwnership                        | 1950            | 1950   | 1950   | 1950   | 94      |
| addTemplate                            | 6268            | 146884 | 149043 | 155543 | 219     |
| addTemplateCategory                    | 1492            | 52540  | 47172  | 77572  | 396     |
| cloneFactory                           | 370             | 1036   | 370    | 2370   | 3       |
| cloneRegistry                          | 348             | 388    | 348    | 2348   | 99      |
| deploy                                 | 807             | 386685 | 460951 | 563031 | 148     |
| nominateNewDependencyOwner             | 2648            | 48588  | 48588  | 94529  | 2       |
| nominateNewOwner                       | 23727           | 23727  | 23727  | 23727  | 94      |
| owner                                  | 2348            | 2348   | 2348   | 2348   | 1       |
| templateRegistry                       | 392             | 432    | 392    | 2392   | 99      |
| toggleTemplateEndorsement              | 619             | 4420   | 3997   | 12224  | 214     |


`DeploymentController.sol` AFTER

|                                        |                 |        |        |        |         |          |
|----------------------------------------|-----------------|--------|--------|--------|---------|----------|
| Deployment Cost                        | Deployment Size |        |        |        |         | diff(avg)|
| 858997                                 | 4820            |        |        |        |         |   +6728  |
| Function Name                          | min             | avg    | median | max    | # calls |          |
| acceptDependencyOwnership              | 6692            | 6692   | 6692   | 6692   | 116     | -338     |
| acceptOwnership                        | 1950            | 1950   | 1950   | 1950   | 94      | 0        |
| addTemplate                            | 6162            | 146358 | 148937 | 153437 | 219     | -526     |
| addTemplateCategory                    | 1389            | 52367  | 47069  | 75469  | 396     | -173     |
| cloneFactory                           | 249             | 249    | 249    | 249    | 3       | -787     |
| cloneRegistry                          | 227             | 227    | 227    | 227    | 99      | -161     |
| deploy                                 | 807             | 385146 | 460639 | 561161 | 148     | -1539    |
| nominateNewDependencyOwner             | 2648            | 45425  | 45425  | 88202  | 2       | -3163    |
| nominateNewOwner                       | 23727           | 23727  | 23727  | 23727  | 94      | 0        |
| owner                                  | 2348            | 2348   | 2348   | 2348   | 1       | 0        |
| templateRegistry                       | 271             | 271    | 271    | 271    | 99      | -161     |
| toggleTemplateEndorsement              | 619             | 4308   | 3894   | 10121  | 214     | -112     |



`VaultController` BEFORE
|                                        |                 |         |         |         |         |
|----------------------------------------|-----------------|---------|---------|---------|---------|
| Deployment Cost                        | Deployment Size |         |         |         |         |
| 4259463                                | 21582           |         |         |         |         |
| Function Name                          | min             | avg     | median  | max     | # calls |
| acceptAdminProxyOwnership              | 1193            | 2491    | 2482    | 4722    | 96      |
| activeTemplateId                       | 527             | 1860    | 2527    | 2527    | 3       |
| addStakingRewardsTokens                | 2131            | 44521   | 14957   | 234223  | 8       |
| addTemplateCategories                  | 2843            | 212165  | 223749  | 223749  | 101     |
| adminProxy                             | 2415            | 2415    | 2415    | 2415    | 1       |
| changeStakingRewardsSpeeds             | 1130            | 10718   | 10243   | 20783   | 3       |
| changeVaultAdapters                    | 197298          | 197298  | 197298  | 197298  | 1       |
| changeVaultFees                        | 10345           | 10345   | 10345   | 10345   | 1       |
| cloneRegistry                          | 439             | 439     | 439     | 439     | 1       |
| deployAdapter                          | 4066            | 385225  | 516004  | 827962  | 15      |
| deployStaking                          | 2613            | 150733  | 10908   | 438701  | 9       |
| deployVault                            | 10521           | 1985961 | 2138926 | 2433475 | 36      |
| deploymentController                   | 373             | 1373    | 1373    | 2373    | 2       |
| escrow                                 | 2414            | 2414    | 2414    | 2414    | 1       |
| fundStakingRewards                     | 1152            | 21480   | 21480   | 41808   | 2       |
| harvestCooldown                        | 429             | 429     | 429     | 429     | 1       |
| nominateNewAdminProxyOwner             | 2651            | 25553   | 33187   | 33187   | 4       |
| owner                                  | 2371            | 2371    | 2371    | 2371    | 1       |
| pauseAdapters                          | 9702            | 50467   | 70850   | 70850   | 6       |
| pauseVaults                            | 9703            | 27781   | 36820   | 36820   | 6       |
| performanceFee                         | 385             | 385     | 385     | 385     | 1       |
| permissionRegistry                     | 2481            | 2481    | 2481    | 2481    | 1       |
| proposeVaultAdapters                   | 971             | 26592   | 11890   | 62325   | 6       |
| proposeVaultFees                       | 820             | 29366   | 9879    | 63128   | 5       |
| setActiveTemplateId                    | 2662            | 5505    | 4815    | 9040    | 3       |
| setAdapterHarvestCooldowns             | 2910            | 14618   | 14618   | 26327   | 2       |
| setAdapterPerformanceFees              | 2820            | 14519   | 14519   | 26218   | 2       |
| setDeploymentController                | 2673            | 7820    | 3785    | 21040   | 4       |
| setEscrowTokenFees                     | 3040            | 17189   | 3077    | 45451   | 3       |
| setHarvestCooldown                     | 2529            | 20609   | 23895   | 25895   | 13      |
| setPerformanceFee                      | 2618            | 20698   | 23984   | 25984   | 13      |
| setPermissions                         | 3012            | 34864   | 42139   | 46493   | 20      |
| templateRegistry                       | 437             | 437     | 437     | 437     | 1       |
| toggleTemplateEndorsements             | 1027            | 10890   | 8389    | 20889   | 210     |
| unpauseAdapters                        | 9680            | 18342   | 9680    | 35666   | 3       |
| unpauseVaults                          | 9615            | 10371   | 9615    | 11883   | 3       |
| vaultRegistry                          | 2459            | 2459    | 2459    | 2459    | 1       |

`VaultController` AFTER
|                                        |                 |         |         |         |         |          |
|----------------------------------------|-----------------|---------|---------|---------|---------|----------|
| Deployment Cost                        | Deployment Size |         |         |         |         | diff(avg)|
| 4444040                                | 23267           |         |         |         |         | +184,577 |
| Function Name                          | min             | avg     | median  | max     | # calls |          |
| acceptAdminProxyOwnership              | 1066            | 2389    | 2380    | 4620    | 96      | -102     |
| activeTemplateId                       | 527             | 1860    | 2527    | 2527    | 3       | 0        |
| addStakingRewardsTokens                | 2131            | 43746   | 14639   | 233608  | 8       | -775     |
| addTemplateCategories                  | 2843            | 211176  | 222913  | 222913  | 101     | -989     |
| adminProxy                             | 294             | 294     | 294     | 294     | 1       | -2121    |
| changeStakingRewardsSpeeds             | 1130            | 10611   | 10137   | 20568   | 3       | -107     |
| changeVaultAdapters                    | 197192          | 197192  | 197192  | 197192  | 1       | -106     |
| changeVaultFees                        | 10239           | 10239   | 10239   | 10239   | 1       | -106     |
| cloneRegistry                          | 439             | 439     | 439     | 439     | 1       | 0        |
| deployAdapter                          | 3860            | 381494  | 514953  | 820493  | 15      | -3731    |
| deployStaking                          | 2407            | 146408  | 10490   | 425753  | 9       | -4325    |
| deployVault                            | 10315           | 1973830 | 2125664 | 2420213 | 36      | -12131   |
| deploymentController                   | 373             | 1373    | 1373    | 2373    | 2       | 0        |
| escrow                                 | 293             | 293     | 293     | 293     | 1       | -2121    |
| fundStakingRewards                     | 1152            | 21440   | 21440   | 41728   | 2       | -40      |
| harvestCooldown                        | 429             | 429     | 429     | 429     | 1       | 0        |
| nominateNewAdminProxyOwner             | 2651            | 23966   | 31072   | 31072   | 4       | -1587    |
| owner                                  | 2371            | 2371    | 2371    | 2371    | 1       | 0        |
| pauseAdapters                          | 9596            | 50290   | 70638   | 70638   | 6       | -177     |
| pauseVaults                            | 9597            | 27604   | 36608   | 36608   | 6       | -177     |
| performanceFee                         | 385             | 385     | 385     | 385     | 1       | 0        |
| permissionRegistry                     | 360             | 360     | 360     | 360     | 1       | -2121    |
| proposeVaultAdapters                   | 971             | 26468   | 11784   | 62113   | 6       | -124     |
| proposeVaultFees                       | 820             | 29239   | 9773    | 62916   | 5       | -127     |
| setActiveTemplateId                    | 2662            | 5505    | 4815    | 9040    | 3       | 0        |
| setAdapterHarvestCooldowns             | 2910            | 14565   | 14565   | 26221   | 2       | -53      |
| setAdapterPerformanceFees              | 2820            | 14466   | 14466   | 26112   | 2       | -53      |
| setDeploymentController                | 2673            | 7758    | 3785    | 20792   | 4       | -62      |
| setEscrowTokenFees                     | 3040            | 15778   | 3077    | 41218   | 3       | -1411    |
| setHarvestCooldown                     | 2529            | 20609   | 23895   | 25895   | 13      | 0        |
| setPerformanceFee                      | 2618            | 20698   | 23984   | 25984   | 13      | 0        | 
| setPermissions                         | 3012            | 32242   | 38906   | 42260   | 20      | -2622    |
| templateRegistry                       | 437             | 437     | 437     | 437     | 1       | 0        |
| toggleTemplateEndorsements             | 1027            | 10273   | 8180    | 18680   | 210     | -617     |
| unpauseAdapters                        | 9574            | 18214   | 9574    | 35496   | 3       | -128     |
| unpauseVaults                          | 9509            | 10243   | 9509    | 11713   | 3       | -128     |
| vaultRegistry                          | 338             | 338     | 338     | 338     | 1       | -2121    |


Total average gas saved for function calls in `DeploymentController.sol`: `6,960`
Total average gas saved for function calls in `VaultController.sol`: `38,162`
