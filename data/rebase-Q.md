Adding test to nominate new admin proxy owner to zero address in `test.vault.VaultController.t.sol`

finding 1
Unnecessary gas spending when nominating to zero address which doesn't affect the change of ownership

```
    function test__nominateZeroAddressNewAdminProxyOwner() public {
        controller.nominateNewAdminProxyOwner(address(0));
        assertEq(adminProxy.nominatedOwner(), address(0));
    }
```

Test run:
```
 ~/se/c/l/2023-01-popcorn │ on main !1 ?1  forge test --match-path test/vault/VaultController.t.sol --match-test "test__nominateNewAdminProxyOwner" -vvvvvv

[⠒] Compiling...
No files changed, compilation skipped

Running 1 test for test/vault/VaultController.t.sol:VaultControllerTest
[PASS] test__nominateNewAdminProxyOwner() (gas: 43681)
Traces:
  [21426979] VaultControllerTest::setUp() 
    ├─ [3209347] → new MultiRewardStaking@0x5615dEB798BB3E4dFa0139dFa1b3D433Cc23b72f
    │   └─ ← 16029 bytes of code
    ├─ [2834812] → new MockAdapter@0x2e234DAe75C793f67A35089C9d99245E1C58470b
    │   └─ ← 14026 bytes of code
    ├─ [145390] → new MockStrategy@0xF62849F9A0B5Bf2913b396098F7c7019b51A820a
    │   └─ ← 726 bytes of code
    ├─ [3170056] → new Vault@0x5991A2dF15A8F6A256D3Ec51E99254Cd3fb576A9
    │   └─ ← 15612 bytes of code
    ├─ [614529] → new MockERC20@0xc7183455a4C133Ae270771860664b6B7ec320bB1
    │   └─ ← 2728 bytes of code
    ├─ [614529] → new MockERC20@0xa0Cb889707d426A7A386870A03bc70d1b0697598
    │   └─ ← 2728 bytes of code
    ├─ [251979] → new AdminProxy@0x1d1499e622D69689cdf9004d05Ec547d650Ff211
    │   ├─ emit OwnerChanged(oldOwner: 0x0000000000000000000000000000000000000000, newOwner: VaultControllerTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496])
    │   └─ ← 1140 bytes of code
    ├─ [395315] → new PermissionRegistry@0xA4AD4f68d0b91CFD19687c881e50f3A00242828c
    │   ├─ emit OwnerChanged(oldOwner: 0x0000000000000000000000000000000000000000, newOwner: AdminProxy: [0x1d1499e622D69689cdf9004d05Ec547d650Ff211])
    │   └─ ← 1856 bytes of code
    ├─ [795315] → new VaultRegistry@0x03A6a84cD762D9707A21605b548aaaB891562aAb
    │   ├─ emit OwnerChanged(oldOwner: 0x0000000000000000000000000000000000000000, newOwner: AdminProxy: [0x1d1499e622D69689cdf9004d05Ec547d650Ff211])
    │   └─ ← 3854 bytes of code
    ├─ [1388216] → new MultiRewardEscrow@0xD6BbDE9174b1CdAa358d2Cf4D57D1a9F7178FBfF
    │   ├─ emit OwnerChanged(oldOwner: 0x0000000000000000000000000000000000000000, newOwner: AdminProxy: [0x1d1499e622D69689cdf9004d05Ec547d650Ff211])
    │   └─ ← 6704 bytes of code
    ├─ [296622] → new CloneFactory@0x15cF58144EF33af1e14b5208015d11F9143E27b9
    │   ├─ emit OwnerChanged(oldOwner: 0x0000000000000000000000000000000000000000, newOwner: VaultControllerTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496])
    │   └─ ← 1363 bytes of code
    ├─ [371696] → new CloneRegistry@0x212224D2F2d262cd093eE13240ca4873fcCBbA3C
    │   ├─ emit OwnerChanged(oldOwner: 0x0000000000000000000000000000000000000000, newOwner: VaultControllerTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496])
    │   └─ ← 1738 bytes of code
    ├─ [918839] → new TemplateRegistry@0x2a07706473244BC757E10F2a9E86fB532828afe3
    │   ├─ emit OwnerChanged(oldOwner: 0x0000000000000000000000000000000000000000, newOwner: VaultControllerTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496])
    │   └─ ← 4471 bytes of code
    ├─ [852269] → new DeploymentController@0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7
    │   ├─ emit OwnerChanged(oldOwner: 0x0000000000000000000000000000000000000000, newOwner: VaultControllerTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496])
    │   └─ ← 3805 bytes of code
    ├─ [23631] CloneFactory::nominateNewOwner(DeploymentController: [0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7]) 
    │   ├─ emit OwnerNominated(newOwner: DeploymentController: [0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7])
    │   └─ ← ()
    ├─ [23711] CloneRegistry::nominateNewOwner(DeploymentController: [0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7]) 
    │   ├─ emit OwnerNominated(newOwner: DeploymentController: [0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7])
    │   └─ ← ()
    ├─ [23713] TemplateRegistry::nominateNewOwner(DeploymentController: [0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7]) 
    │   ├─ emit OwnerNominated(newOwner: DeploymentController: [0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7])
    │   └─ ← ()
    ├─ [7030] DeploymentController::acceptDependencyOwnership() 
    │   ├─ [1879] CloneFactory::acceptOwnership() 
    │   │   ├─ emit OwnerChanged(oldOwner: VaultControllerTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496], newOwner: DeploymentController: [0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7])
    │   │   └─ ← ()
    │   ├─ [1932] CloneRegistry::acceptOwnership() 
    │   │   ├─ emit OwnerChanged(oldOwner: VaultControllerTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496], newOwner: DeploymentController: [0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7])
    │   │   └─ ← ()
    │   ├─ [1932] TemplateRegistry::acceptOwnership() 
    │   │   ├─ emit OwnerChanged(oldOwner: VaultControllerTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496], newOwner: DeploymentController: [0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7])
    │   │   └─ ← ()
    │   └─ ← ()
    ├─ [23727] DeploymentController::nominateNewOwner(AdminProxy: [0x1d1499e622D69689cdf9004d05Ec547d650Ff211]) 
    │   ├─ emit OwnerNominated(newOwner: AdminProxy: [0x1d1499e622D69689cdf9004d05Ec547d650Ff211])
    │   └─ ← ()
    ├─ [2920] AdminProxy::execute(DeploymentController: [0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7], 0x79ba509700000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000000) 
    │   ├─ [1950] DeploymentController::acceptOwnership() 
    │   │   ├─ emit OwnerChanged(oldOwner: VaultControllerTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496], newOwner: AdminProxy: [0x1d1499e622D69689cdf9004d05Ec547d650Ff211])
    │   │   └─ ← ()
    │   └─ ← true, 0x
    ├─ [4259463] → new VaultController@0xD16d567549A2a2a2005aEACf7fB193851603dd70
    │   ├─ emit OwnerChanged(oldOwner: 0x0000000000000000000000000000000000000000, newOwner: VaultControllerTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496])
    │   ├─ emit DeploymentControllerChanged(oldController: 0x0000000000000000000000000000000000000000, newController: DeploymentController: [0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7])
    │   ├─ [348] DeploymentController::cloneRegistry() [staticcall]
    │   │   └─ ← CloneRegistry: [0x212224D2F2d262cd093eE13240ca4873fcCBbA3C]
    │   ├─ [392] DeploymentController::templateRegistry() [staticcall]
    │   │   └─ ← TemplateRegistry: [0x2a07706473244BC757E10F2a9E86fB532828afe3]
    │   └─ ← 20138 bytes of code
    ├─ [23666] AdminProxy::nominateNewOwner(VaultController: [0xD16d567549A2a2a2005aEACf7fB193851603dd70]) 
    │   ├─ emit OwnerNominated(newOwner: VaultController: [0xD16d567549A2a2a2005aEACf7fB193851603dd70])
    │   └─ ← ()
    ├─ [2482] VaultController::acceptAdminProxyOwnership() 
    │   ├─ [1896] AdminProxy::acceptOwnership() 
    │   │   ├─ emit OwnerChanged(oldOwner: VaultControllerTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496], newOwner: VaultController: [0xD16d567549A2a2a2005aEACf7fB193851603dd70])
    │   │   └─ ← ()
    │   └─ ← ()
    ├─ [223749] VaultController::addTemplateCategories([0x5661756c74000000000000000000000000000000000000000000000000000000, 0x4164617074657200000000000000000000000000000000000000000000000000, 0x5374726174656779000000000000000000000000000000000000000000000000, 0x5374616b696e6700000000000000000000000000000000000000000000000000]) 
    │   ├─ [70281] AdminProxy::execute(DeploymentController: [0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7], 0xa3f834495661756c74000000000000000000000000000000000000000000000000000000) 
    │   │   ├─ [69072] DeploymentController::addTemplateCategory(0x5661756c74000000000000000000000000000000000000000000000000000000) 
    │   │   │   ├─ [68187] TemplateRegistry::addTemplateCategory(0x5661756c74000000000000000000000000000000000000000000000000000000) 
    │   │   │   │   ├─ emit TemplateCategoryAdded(templateCategory: 0x5661756c74000000000000000000000000000000000000000000000000000000)
    │   │   │   │   └─ ← ()
    │   │   │   └─ ← ()
    │   │   └─ ← true, 0x
    │   ├─ [48381] AdminProxy::execute(DeploymentController: [0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7], 0xa3f834494164617074657200000000000000000000000000000000000000000000000000) 
    │   │   ├─ [47172] DeploymentController::addTemplateCategory(0x4164617074657200000000000000000000000000000000000000000000000000) 
    │   │   │   ├─ [46287] TemplateRegistry::addTemplateCategory(0x4164617074657200000000000000000000000000000000000000000000000000) 
    │   │   │   │   ├─ emit TemplateCategoryAdded(templateCategory: 0x4164617074657200000000000000000000000000000000000000000000000000)
    │   │   │   │   └─ ← ()
    │   │   │   └─ ← ()
    │   │   └─ ← true, 0x
    │   ├─ [48381] AdminProxy::execute(DeploymentController: [0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7], 0xa3f834495374726174656779000000000000000000000000000000000000000000000000) 
    │   │   ├─ [47172] DeploymentController::addTemplateCategory(0x5374726174656779000000000000000000000000000000000000000000000000) 
    │   │   │   ├─ [46287] TemplateRegistry::addTemplateCategory(0x5374726174656779000000000000000000000000000000000000000000000000) 
    │   │   │   │   ├─ emit TemplateCategoryAdded(templateCategory: 0x5374726174656779000000000000000000000000000000000000000000000000)
    │   │   │   │   └─ ← ()
    │   │   │   └─ ← ()
    │   │   └─ ← true, 0x
    │   ├─ [48381] AdminProxy::execute(DeploymentController: [0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7], 0xa3f834495374616b696e6700000000000000000000000000000000000000000000000000) 
    │   │   ├─ [47172] DeploymentController::addTemplateCategory(0x5374616b696e6700000000000000000000000000000000000000000000000000) 
    │   │   │   ├─ [46287] TemplateRegistry::addTemplateCategory(0x5374616b696e6700000000000000000000000000000000000000000000000000) 
    │   │   │   │   ├─ emit TemplateCategoryAdded(templateCategory: 0x5374616b696e6700000000000000000000000000000000000000000000000000)
    │   │   │   │   └─ ← ()
    │   │   │   └─ ← ()
    │   │   └─ ← true, 0x
    │   └─ ← ()
    ├─ [147043] DeploymentController::addTemplate(0x5374616b696e6700000000000000000000000000000000000000000000000000, 0x4d756c74695265776172645374616b696e670000000000000000000000000000, (0x5615dEB798BB3E4dFa0139dFa1b3D433Cc23b72f, false, cid, true, 0x26097A3BC5814e69CA3eC555c4E4e19d23E902bd, [0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000])) 
    │   ├─ [143763] TemplateRegistry::addTemplate(0x5374616b696e6700000000000000000000000000000000000000000000000000, 0x4d756c74695265776172645374616b696e670000000000000000000000000000, (0x5615dEB798BB3E4dFa0139dFa1b3D433Cc23b72f, false, cid, true, 0x26097A3BC5814e69CA3eC555c4E4e19d23E902bd, [0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000])) 
    │   │   ├─ emit TemplateAdded(templateCategory: 0x5374616b696e6700000000000000000000000000000000000000000000000000, templateId: 0x4d756c74695265776172645374616b696e670000000000000000000000000000, implementation: MultiRewardStaking: [0x5615dEB798BB3E4dFa0139dFa1b3D433Cc23b72f])
    │   │   └─ ← ()
    │   └─ ← ()
    ├─ [8389] VaultController::toggleTemplateEndorsements([0x5374616b696e6700000000000000000000000000000000000000000000000000], [0x4d756c74695265776172645374616b696e670000000000000000000000000000]) 
    │   ├─ [5209] AdminProxy::execute(DeploymentController: [0x3D7Ebc40AF7092E3F1C81F2e996cbA5Cae2090d7], 0x560811fa5374616b696e67000000000000000000000000000000000000000000000000004d756c74695265776172645374616b696e670000000000000000000000000000) 
    │   │   ├─ [3997] DeploymentController::toggleTemplateEndorsement(0x5374616b696e6700000000000000000000000000000000000000000000000000, 0x4d756c74695265776172645374616b696e670000000000000000000000000000) 
    │   │   │   ├─ [3000] TemplateRegistry::toggleTemplateEndorsement(0x5374616b696e6700000000000000000000000000000000000000000000000000, 0x4d756c74695265776172645374616b696e670000000000000000000000000000) 
    │   │   │   │   ├─ emit TemplateEndorsementToggled(templateCategory: 0x5374616b696e6700000000000000000000000000000000000000000000000000, templateId: 0x4d756c74695265776172645374616b696e670000000000000000000000000000, oldEndorsement: false, newEndorsement: true)
    │   │   │   │   └─ ← ()
    │   │   │   └─ ← ()
    │   │   └─ ← true, 0x
    │   └─ ← ()
    └─ ← ()

  [43681] VaultControllerTest::test__nominateNewAdminProxyOwner() 
    ├─ [33187] VaultController::nominateNewAdminProxyOwner(0x000000000000000000000000000000000000DCbA) 
    │   ├─ [25666] AdminProxy::nominateNewOwner(0x000000000000000000000000000000000000DCbA) 
    │   │   ├─ emit OwnerNominated(newOwner: 0x000000000000000000000000000000000000DCbA)
    │   │   └─ ← ()
    │   └─ ← ()
    ├─ [359] AdminProxy::nominatedOwner() [staticcall]
    │   └─ ← 0x000000000000000000000000000000000000DCbA
    └─ ← ()

Test result: ok. 1 passed; 0 failed; finished in 7.56ms
 ~/se/currents/latest/2023-01-popcorn │ on main !1 ?1                                                                                    ✔ │ at 23:29:59 
```