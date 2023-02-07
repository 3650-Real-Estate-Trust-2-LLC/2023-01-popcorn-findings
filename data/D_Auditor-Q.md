[QA-01]: check that owner == to adminProxy in the Vault Controller contract. external calls may revert because of a difference in owner address (https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol)

[QA-02]: Zero Address check in Vault.sol contract for the adapter is missing. The contract should also check that the adapter != to the vault that is being deployed. (https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)

[QA-03]: 365.25 days is not exactly equal to 365 days in seconds. Hardcode 31, 536, 000 into SECONDS_PER_YEAR in MultiRewardStaking.sol contract (https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)