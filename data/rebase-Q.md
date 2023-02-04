Adding test to nominate new admin proxy owner to zero address in `test.vault.VaultController.t.sol`

finding #1
Unnecessary gas spending when nominating to zero address which doesn't affect the change of ownership

```
    function test__nominateZeroAddressNewAdminProxyOwner() public {
        controller.nominateNewAdminProxyOwner(address(0));
        assertEq(adminProxy.nominatedOwner(), address(0));
    }
```

Test Output for finding #1:
```
 ~/se/currents/latest/2023-01-popcorn │ on main !1 ?1  forge test --match-path test/vault/VaultController.t.sol --match-test "test__nominateZeroAddressNewAdminProxyOwner" -vvvv
[⠔] Compiling...
No files changed, compilation skipped

Running 1 test for test/vault/VaultController.t.sol:VaultControllerTest
[PASS] test__nominateZeroAddressNewAdminProxyOwner() (gas: 21557)
Traces:
  [21557] VaultControllerTest::test__nominateZeroAddressNewAdminProxyOwner() 
    ├─ [13287] VaultController::nominateNewAdminProxyOwner(0x0000000000000000000000000000000000000000) 
    │   ├─ [5766] AdminProxy::nominateNewOwner(0x0000000000000000000000000000000000000000) 
    │   │   ├─ emit OwnerNominated(newOwner: 0x0000000000000000000000000000000000000000)
    │   │   └─ ← ()
    │   └─ ← ()
    ├─ [359] AdminProxy::nominatedOwner() [staticcall]
    │   └─ ← 0x0000000000000000000000000000000000000000
    └─ ← ()

Test result: ok. 1 passed; 0 failed; finished in 4.21ms
```
finding #2
Ownership change implicitly resets performance fees, which can lead to some lost gains if the assumption that vaultcontroller is carrying over existing settings/configs.
PoC: 
Add following test to `test.vault.VaultController.t.sol`:
```
function testFail_ownershipChangeKeepsPerformanceFees() public {      
        uint256 performanceFee = 1e16;
        address[] memory targets = new address[](1);
        addTemplate("Adapter", templateId, adapterImpl, true, true);
        address adapter = deployAdapter();
        targets[0] = adapter;

        // set the fees    
        controller.setPerformanceFee(performanceFee);
        controller.setAdapterPerformanceFees(targets);

        assertEq(controller.performanceFee(), performanceFee);      
        assertEq(IAdapter(adapter).performanceFee(), 1e16);

        VaultController bobController = new VaultController( 
            address(bob), 
            adminProxy, 
            deploymentController,
            vaultRegistry, 
            permissionRegistry, 
            escrow
            );
        
        controller.nominateNewAdminProxyOwner(address(bobController)); 
        assertEq(adminProxy.nominatedOwner(), address(bobController)); 

        bobController.acceptAdminProxyOwnership(); 
        assertEq(adminProxy.owner(), address(bobController));
        
        // bob's performanceFee becomes zero
        assertEq(bobController.performanceFee(), performanceFee);                           
        }
 ```
Test Output for finding #2:
```
 ~/se/c/latest/2023-01-popcorn │ on main !1 ?1  forge test --match-path test/vault/VaultController.t.sol --match-test "testFail_ownershipChangeKeepsPerformanceFees"
[⠒] Compiling...
No files changed, compilation skipped

Running 1 test for test/vault/VaultController.t.sol:VaultControllerTest
[PASS] testFail_ownershipChangeKeepsPerformanceFees() (gas: 4385580)
Logs:
  Error: a == b not satisfied [uint]
    Expected: 10000000000000000
      Actual: 0

Test result: ok. 1 passed; 0 failed; finished in 7.56ms
```