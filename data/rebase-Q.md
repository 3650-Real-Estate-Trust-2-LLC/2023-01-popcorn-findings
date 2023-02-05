Should not nominate new admin proxy to address zero:
```
   function test__nominateZeroAddressNewAdminProxyOwner() public {
        controller.nominateNewAdminProxyOwner(address(0));
        assertEq(adminProxy.nominatedOwner(), address(0));
    }
```

 Test output:
```
 ~/latest/2023-01-popcorn │ on main !1 ?1  forge test --match-path test/vault/VaultController.t.sol --match-test "test__nominateZeroAddressNewAdminProxyOwner" -vvvv                              ✔ │ took 8s │ at 23:48:33 
[⠒] Compiling...
No files changed, compilation skipped

Running 1 test for test/vault/VaultController.t.sol:VaultControllerTest
[PASS] test__nominateZeroAddressNewAdminProxyOwner() (gas: 21579)
Traces:
  [21579] VaultControllerTest::test__nominateZeroAddressNewAdminProxyOwner() 
    ├─ [13287] VaultController::nominateNewAdminProxyOwner(0x0000000000000000000000000000000000000000) 
    │   ├─ [5766] AdminProxy::nominateNewOwner(0x0000000000000000000000000000000000000000) 
    │   │   ├─ emit OwnerNominated(newOwner: 0x0000000000000000000000000000000000000000)
    │   │   └─ ← ()
    │   └─ ← ()
    ├─ [359] AdminProxy::nominatedOwner() [staticcall]
    │   └─ ← 0x0000000000000000000000000000000000000000
    └─ ← ()

Test result: ok. 1 passed; 0 failed; finished in 7.64ms
```
