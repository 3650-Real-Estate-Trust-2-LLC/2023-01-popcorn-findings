Low QA Popcorn contest

[L-01] Use latest Solidity version with a stable pragma statement.
	Using a floating pragma ^0.8.15 statement is discouraged as code can compile to different 
	bytecodes with different compiler versions. 
	Use a stable pragma statement to get a deterministic bytecode. Also use latest Solidity 
	version to get all compiler features, bugfixes and optimizations
	
[L-02]The Emit to be after the action 
	https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L544-L545
	File: src/vault/Vault.sol
	## Proof of Concept	
	
    emit ChangedFees(fees, proposedFees);
    fees = proposedFees;
				
	## Recommended Mitigation Steps:
	emit ChangedFees(fees, proposedFees);
    fees = proposedFees;
	
	https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L556-L558
	## Proof of Concept	
	emit FeeRecipientUpdated(feeRecipient, _feeRecipient);

    feeRecipient = _feeRecipient;
		
    ## Recommended Mitigation Steps:
	feeRecipient = _feeRecipient;	
	emit FeeRecipientUpdated(feeRecipient, _feeRecipient);
	https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L606-L608
	## Proof of Concept	
		emit ChangedAdapter(adapter, proposedAdapter);

		adapter = proposedAdapter
	## Recommended Mitigation Steps:
	adapter = proposedAdapter
	emit ChangedAdapter(adapter, proposedAdapter);
		
	
[L-03]	
================================================================================================
Non-Critical
[NC-01]MISSING NATSPEC
	File: src/utils/EIP165.sol
	https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/EIP165.sol#L7


[N-02] INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS
	Context:
	All Contracts without Vault.sol and AdapterBase.sol

	Description:
	https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

	If Return parameters are declared, you must prefix them with /// @return.

	Some code analysis programs do analysis by reading NatSpec details, if they can’t see the “@return” tag, they do incomplete analysis.

	Recommendation:
	Include return parameters in NatSpec comments

	Recommendation Code Style:
	/// @notice information about what a function does
	   /// @param pageId The id of the page to get the URI for.
	   /// @return Returns a page's URI if it has been minted 
	   function tokenURI(uint256 pageId) public view virtual override returns (string memory) {
		   if (pageId == 0 || pageId > currentId) revert("NOT_MINTED");
		   return string.concat(BASE_URI, pageId.toString());
	   }
	   
================================================================================================
QA
[QA-01]NOT USING THE LATEST VERSION OF OPENZEPPELIN BECAUSE OF DEPENDENCIES
	The configuration file package.json says the project uses 4.8.0 of OpenZeppelin which is not latest updated version
	https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/package.json#L25-L26
	The last version is v4.8.1