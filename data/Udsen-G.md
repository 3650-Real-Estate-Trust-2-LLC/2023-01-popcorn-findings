## 1. CALLDATA VARIABLE SHOULD BE USED TO EMIT THE EVENT INSTEAD OF USING THE STATE VARIABLE, TO SAVE GAS.

It is gas efficient to use the calldata variable as the argument to `QuitPeriodSet` event instead of using the state variable. 
This will save an extra `SLOAD`.

    function setQuitPeriod(uint256 _quitPeriod) external onlyOwner {
        if (_quitPeriod < 1 days || _quitPeriod > 7 days)
            revert InvalidQuitPeriod();

        quitPeriod = _quitPeriod;

        emit QuitPeriodSet(quitPeriod); 
    }

As shown above `quitPeriod` is a state variable which is used as the argument to emit the event.	
The `emit QuitPeriodSet(quitPeriod)` event can be coded as follows to save gas:

        emit QuitPeriodSet(_quitPeriod);
		
`_quitPeriod` is calldata variable which is passed into the function as an input parameter.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L629-L636	