###  No same value input control
```
Vault.sol
    function setFeeRecipient(address _feeRecipient) external onlyOwner {
        if (_feeRecipient == address(0)) revert InvalidFeeRecipient();

        emit FeeRecipientUpdated(feeRecipient, _feeRecipient);

        feeRecipient = _feeRecipient;
    }
```
Recommended Mitigation Steps
Add code like this:
``` if (feeRecipient == _feeRecipient revert ADDRESS_SAME();```