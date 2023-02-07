# Beefy Adapter Cannot Be Paused if the Underlying Balance is Equal to Zero

pause( ) in AdapterBase calls _protocolWithdraw( ).  

        function pause() external onlyOwner {
            _protocolWithdraw(totalAssets(), totalSupply());
            // Update the underlying balance to prevent inflation attacks
            underlyingBalance = 0;
            _pause();
         }

However, beefyBooster will revert if the withdrawal amount is ever 0

        function _protocolWithdraw(uint256, uint256 shares)
            internal
            virtual
            override
        {
            uint256 beefyShares = convertToUnderlyingShares(0, shares);
            if (address(beefyBooster) != address(0))
                beefyBooster.withdraw(beefyShares);
            beefyVault.withdraw(beefyShares);
        }

Therefore, the beefy adapter cannot be paused if the underlying balance is ever 0.

## Proof of Concept

    --- a/test/vault/integration/beefy/BeefyAdapter.t.sol
    +++ b/test/vault/integration/beefy/BeefyAdapter.t.sol
    @@ -169,6 +169,13 @@ contract BeefyAdapterTest is AbstractAdapterTest {
 
    +  function test_pause() public {
    +
    +      vm.expectRevert("Cannot withdraw 0");
    +      adapter.pause();
    +
    +  }

## Recommended Mitigation Steps

Have _protocolWithdraw( ) stop execution before calling beefyBooster.withdraw() if beefShares == 0 

        function _protocolWithdraw(uint256, uint256 shares)
            internal
            virtual
            override
        {
            uint256 beefyShares = convertToUnderlyingShares(0, shares);

            if (beefyShares == 0) return;

            if (address(beefyBooster) != address(0))
                beefyBooster.withdraw(beefyShares);
            beefyVault.withdraw(beefyShares);
        }

# Yearn Adapter Cannot Be Paused if the Underlying Balance is Equal to Zero

pause( ) in AdapterBase calls _protocolWithdraw( ).  

        function pause() external onlyOwner {
            _protocolWithdraw(totalAssets(), totalSupply());
            // Update the underlying balance to prevent inflation attacks
            underlyingBalance = 0;
            _pause();
         }

_protocolWithdraw( ) in YearnAdapter calls ConvertToUnderlyingShares, which will throw an arithmetic error if the balance is 0.  
