public should be external
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L197
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L177
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L130
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L111
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L90
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/strategy/Pool2SingleAssetCompounder.sol#L14
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/strategy/Pool2SingleAssetCompounder.sol#L32
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/strategy/Pool2SingleAssetCompounder.sol#L44


a += b  cost more than a = a +b
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L158

saves gas by summing rewardTokens[i] value to local variable and then outside loop 
could remove one of the two functions,in fact they do the same thing and return the same;Could use just one of those:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L404-L412
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/adapter/abstracts/AdapterBase.sol#L419-L421


