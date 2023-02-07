1) anyone can call harvest function inside strategy/Pool2SingleAssetCompounder and this result in a cooldown bypass of the harvestCooldown period of time required between one harvest and another
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/strategy/Pool2SingleAssetCompounder.sol#L44-L67
could fix it by checking caller to be AdapterBase contract only

2) possible Dos gas block problem on this parts,could result in a broken code
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/strategy/Pool2SingleAssetCompounder.sol#L56-L64
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/strategy/Pool2SingleAssetCompounder.sol#L38-L40
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/strategy/Pool2SingleAssetCompounder.sol#L24-L29


3) the "Harvested" event inside AdapterBase.sol should be called only if "harvest()" call has been successfull by handling the return of the delegate call by (bool success) = ... and check if success==true emit Harvested()
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L444-L449

4) lastHarvest variable inside AdapterBase will never be updated after a successful harvest,this will create big problems related to all the harvest function inside AdapterBase by making this function unusable,in fact only the first harvest will work and the following ones wont,can resolve this problem by updating lastHarvest = block.timestamp after successfull harvest
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L441

5) should add important params to Harvested event inside AdapterBase like ID of the harvest or the timestamp
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L431

6) setHarvestCooldown should call harvest before changing harvestCooldown so as not to create future problems with harvest
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L500-L507

7) missing Reentrancy guard to the withdraw function inside AdapterBase.sol
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L210-L235

8) a blacklist system should be added which protects the main functions

9) insufficient coverage of the code,try to raise it up to 100%

10) NATSPEC comments should be increased

11) "_convertToShares" the return could result in a loss of precision due to rounding and damage the deposit function
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L380-L393

12) "_deposit" function call safeTransferFrom without asking for allowance for the tokens,this will result in a revert of the txn
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L153

13) use bytes.concat() instead of abi.encodePacked()
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L648

14) permit function doesn't check for spender==address(0)
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L634

15) should check that receiver is not address(0) in redeem,withdraw,mint and deposit function
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L110
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L129
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L173
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L193
