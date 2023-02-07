fees can be high at moment of deploy

run in VaultController.t.sol

      function test__deployVault_fee() public {
        addTemplate("Adapter", templateId, adapterImpl, true, true);
        addTemplate("Strategy", "MockStrategy", strategyImpl, false, true);
        addTemplate("Vault", "V1", vaultImpl, true, true);
        controller.setPerformanceFee(uint256(1000));
        controller.setHarvestCooldown(1 days);
        rewardToken.mint(address(this), 10 ether);
        rewardToken.approve(address(controller), 10 ether);

        swapTokenAddresses[0] = address(0x9999);
        address adapterClone = 0xD6C5fA22BBE89db86245e111044a880213b35705;
        address strategyClone = 0xe8a41C57AB0019c403D35e8D54f2921BaE21Ed66;
        address stakingClone = 0xE64C695617819cE724c1d35a37BCcFbF5586F752;

        uint256 callTimestamp = block.timestamp;
        address vaultClone = controller.deployVault(
            VaultInitParams({
                asset: iAsset,
                adapter: IERC4626(address(0)),
                fees: VaultFees({
                    deposit: 8 ether,//fees
                    withdrawal:8 ether,//fees
                    management: 8 ether,//fees
                    performance: 8 ether//fees
                }),
                feeRecipient: feeRecipient,
                owner: address(this)
            }),
            DeploymentArgs({id: templateId, data: abi.encode(uint256(100))}),
            DeploymentArgs({id: "MockStrategy", data: ""}),
            address(0),
            abi.encode(
                address(rewardToken),
                0.1 ether,
                1 ether,
                true,
                10000000,
                2 days,
                1 days
            ),
            VaultMetadata({
                vault: address(0),
                staking: address(0),
                creator: address(this),
                metadataCID: metadataCid,
                swapTokenAddresses: swapTokenAddresses,
                swapAddress: address(0x5555),
                exchange: uint256(1)
            }),
            0
        );
      
        console.log("fees deposit    :%d",IVault(vaultClone).fees().deposit);
        console.log("fees withdrawal :%d",IVault(vaultClone).fees().withdrawal);
        console.log("fees management :%d",IVault(vaultClone).fees().management);
        console.log("fees performance:%d",IVault(vaultClone).fees().performance);
    
       
    }

result 

     Running 1 test for test/vault/VaultController.t.sol:VaultControllerTest
     [PASS] test__deployVault_fee() (gas: 2917592)
     Logs:
     fees deposit    :8000000000000000000
     fees withdrawal :8000000000000000000
     fees management :8000000000000000000
     fees performance:8000000000000000000

   Test result: ok. 1 passed; 0 failed; finished in 31.76ms

