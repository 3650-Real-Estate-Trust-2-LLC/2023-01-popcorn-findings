##  [LOW-1] initialize() function can be called by anybody
initialize() function can be called anybody when the contract is not initialized.

More importantly, if someone else runs this function, they will have full authority because of the  initialize() function. Also, there is no 0 address check in the address arguments of the initialize() function, which must be defined.

Here is a definition of initialize() function.
consider validation check

https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L46-L84
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L41-L57
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L57-L98
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L34-L55

### Recommended Mitigation Steps

``` solidity
if (msg.sender != DEPLOYER_ADDRESS) {
   revert NotDeployer();
}
```

##  [LOW-2]  Lack of zero address check
Adding non-zero address checks on the following function's parameters can help ensure the ownership of contracts is not lost or the contracts do not need to be redeployed if any of them is provided as zero accidentally.

``` solidity
File:  quest\AdminProxy.sol
16:   constructor(address _owner) Owned(_owner) {}
File:  quest\CloneFactory.sol
23:   constructor(address _owner) Owned(_owner) {}
File:  quest\PermissionRegistry.sol
20:   constructor(address _owner) Owned(_owner) {}
File:  quest\TemplateRegistry.sol
24:   constructor(address _owner) Owned(_owner) {}
File:  quest\VaultController.sol
61:     adminProxy = _adminProxy;
62:     vaultRegistry = _vaultRegistry;
63:     permissionRegistry = _permissionRegistry;
64:     escrow = _escrow;
File:  quest\VaultRegistry.sol
21:   constructor(address _owner) Owned(_owner) {}
File:  quest\DeploymentController.sol
41:     cloneFactory = _cloneFactory;
42:     cloneRegistry = _cloneRegistry;
43:     templateRegistry = _templateRegistry;
File:  quest\MultiRewardEscrow.sol
31:     feeRecipient = _feeRecipient;
```

### Recommended Mitigation Steps
Consider adding non-zero address checks on the parameters.

##  [LOW-3] Consider  implmentation logic  maxMint & maxDeposit
Return the maximum amount of shares that mint would allow to be deposited to the receiver without causing a revert, which must not be higher than the actual maximum that would be accepted (it should underestimate if necessary)

### Recommended Mitigation Steps
maxMint() and maxDeposit() should reflect the limitation of maxSupply.

Consider changing maxMint() and maxDeposit() to:

``` solidity
function maxMint(address) public view virtual returns (uint256) {
    if (totalSupply >= maxSupply) {
        return 0;
    }
    return maxSupply - totalSupply;
}
function maxDeposit(address) public view virtual returns (uint256) {
    return convertToAssets(maxMint(address(0)));
}
```

## [L-4] Lack of Input Validation
For defence-in-depth purpose, it is recommended to perform additional validation against the amount that the user is attempting to deposit, mint, withdraw and redeem to ensure that the submitted amount is valid.

### Recommended Mitigation Steps
```  solidity
File: c:\Users\pc\Desktop\quest\Vault.sol
134: function deposit(uint256 assets, address receiver)
135:         public
136:         nonReentrant
137:         whenNotPaused
138:         syncFeeCheckpoint
139:         returns (uint256 shares)
140:     {
+  require(assets <= maxDeposit(receiver), "deposit more than max");
```

``` solditiy
File: c:\Users\pc\Desktop\quest\Vault.sol
170:    function mint(uint256 shares, address receiver)
171:         public
172:         nonReentrant
173:         whenNotPaused
174:         syncFeeCheckpoint
175:         returns (uint256 assets)
176:     {

+   require(shares <= maxMint(receiver), "mint more than max");
```
``` solidity
File: c:\Users\pc\Desktop\quest\Vault.sol
211:     function withdraw(
212:         uint256 assets,
213:         address receiver,
214:         address owner
215:     ) public nonReentrant syncFeeCheckpoint returns (uint256 shares) {
+  require(assets <= maxWithdraw(owner), "withdraw more than max");
```

``` solidity
File: c:\Users\pc\Desktop\quest\Vault.sol
253:  function redeem(
254:         uint256 shares,
255:         address receiver,
256:         address owner
257:     ) public nonReentrant returns (uint256 assets) {
258:    +    require(shares <= maxRedeem(owner), "redeem more than max");
```

## [L-5]  More accurate to use <= for validity of timestamp changeFees
Currently, the check for changeFees timestamp to not exceed block.timestamp is done with a strict comparison. Using <= can be better here as VALID_FOR implies that the changeFees timestamp would be valid for the entire duration.
``` solidity
    function changeFees() external {
        if (block.timestamp < proposedFeeTime + quitPeriod)
            revert NotPassedQuitPeriod(quitPeriod);

        emit ChangedFees(fees, proposedFees);
        fees = proposedFees;
    }
```

### Recommendation
Change to `block.timestamp <= proposedFeeTime + quitPeriod`