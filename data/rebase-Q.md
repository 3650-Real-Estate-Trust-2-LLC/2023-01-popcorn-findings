Adding test to nominate new admin proxy owner to zero address in `test.vault.VaultController.t.sol`

finding #1
Vault ownership change implicitly resets performance fees, which can lead to some lost gains if the assumption that vaultcontroller is carrying over existing settings/configs.
PoC: 
Add the following test to `test.vault.VaultController.t.sol`:
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