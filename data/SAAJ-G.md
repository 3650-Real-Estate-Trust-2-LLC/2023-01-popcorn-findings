
# Gas Optimizations Report

This report focuses on Popcorn contest, in context of various improvements that can be made in terms of gas cost.

Some of the opportunities identified for improving gas efficiency throughout the codebase of Popcorn Protocol are categorised into 07 main areas; with further multiple instances in each of the category.


# Summary

[G-01] State variable values are by default internal (13 Instance)
[G-02] Immutable has more gas efficiency than constant (03 Instances)
[G-03] Multiple mappings can be combined into a single one (17 Instances)
[G-04] Uint/Int lower than 32 bytes incurs overhead (75 Instances)
[G-05] Wastage of deployed gas when return is not present for returns function (08 Instances)
[G-06] 0perator assignment is more gas efficient than compound assignment (12 Instances)
[G-07] Functions with public visibility, if not called within the contract needed to be changed external (07 Instances) 
# [G-01] State variable values are by default internal (13 Instance)

When visibility for a state variable is not specified, the default value is already internal.

Declaring visibility of the variable internal consumes extra gas which is not needed.

Link to the Code:
1.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L21
2.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L22
3.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L71
4.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L24
5.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L25
6.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L438
7.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L439
8.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L38
9.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L657
10.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L658
11.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L40
12.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L38
13.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L241


# [G-02] Immutable has more gas efficiency than constant (03 Instances)

Using immutable instead of constant, save more gas due to avoiding storage access for state variables.

Variable values are set through constructor when using immutable, which also eliminates the need of assigning initial values to state variable making them more efficient in terms of gas cost

Link to the Code:
1.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L25
2.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L31
3.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L35


# [G-03] Multiple mappings can be combine into a single one (17 Instances)

When multiple mappings are used in same function, it’s better to combined them into a single mapping of an address struct.

Combined mapping reduces storage slot per mapping and also are cheaper in terms of associated Stack operations calculation carried out.

Link to the Code:
1.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L28
2.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L30
3.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultRegistry.sol#L28
4.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultRegistry.sol#L30
5.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L31
6.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L32
7.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L33
8.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L35
9.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L64
10.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L67
11.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L69
12.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L194
13.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L211
14.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L213
15.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L216
16.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L218
17.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L440



# [G-04] Uint/Int lower than 32 bytes consumes incurs overhead (75 Instances)

Contract gas usage increases as EVM standard operation are of 32 bytes. If any element is smaller than 32 bytes (i.e.; 256 bits) it will cause EVM to consume more gas which can be around 12 gas depending on size for reducing the size to given output like uint8.


Link to the Code:

1.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L92
2.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L93
3.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L33
4.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L171
5.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L220
6.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L245
7.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L246
8.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L248
9.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L249
10.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L250
11.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L274
12.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L275
13.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L296
14.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L307
15.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L308
16.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L340
17.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L352
18.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L353
19.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L354
20.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L373
21.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L404
22.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L450
23.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L38
24.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L100
25.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L668
26.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L314
27.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L336
28.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L337
29.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L353
30.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L357
31.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L373
32.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L374
33.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L436
34.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L437
35.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L440
36.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L443
37.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L444
38.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L446
39.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L486
40.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L488
41.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L517
42.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L563
43.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L583
44.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L606
45.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L619
46.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L632
47.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L645
48.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L765
49.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L805
50.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L38
51.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L93
52.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L637
53.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardEscrow.sol#L11
54.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardEscrow.sol#L13
55.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardEscrow.sol#L15
56.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardEscrow.sol#L36
57.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardEscrow.sol#L37
58.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L16
59.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L18
60.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L21
61.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L23
62.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L25
63.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L30
64.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L32
65.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L34
66.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L40
67.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L43
68.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L44
69.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L45
70.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardStaking.sol#L48
71.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVault.sol#L9
72.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVault.sol#L10
73.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVault.sol#L11
74.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVault.sol#L12
75.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/vault/IVaultController.sol#L60


# [G-05] Wastage of deployed gas when return is not present for returns function (08 Instances)

Wastage of gas during deployment; when return is absent for named variable when function returns.

Link to the Code:
1.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L326
2.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L340
3.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L358
4.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L380
5.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L97
6.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L122
7.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L258
8.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L286


# [G-06] 0perator assignment is more gas efficient than compound assignment (12 Instances)

Compound assignment operators (+= / -=) are more expensive in terms of gas consumption and needs to be avoided.

Operator assignments (a = a + b / a - b) are preferable in terms of gas optimization.

Link to the Code:
1.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L110
2.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L162
3.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L178
4.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L357
5.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L408
6.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L431
7.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L228
8.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L343
9.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L365
10.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L387
11.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L158
12.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L225



# [G-07] Functions with public visibility, if not called within the contract needed to be changed external (07 Instances)

Functions with public visibility, if not called within the contract needed to be changed external.

Link to the Code:
1.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L144
2.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L148
3.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L167
4.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L221
5.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L230
6.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L445
7.	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L664