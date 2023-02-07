## 1. `abi.encodePacked()` SHOULD NOT BE USED WITH DYNAMIC TYPES WHEN PASSING THE RESULT TO A HASH FUNCTION SUCH AS `keccak256()`

        abi.encodePacked("Popcorn", name(), block.timestamp, "Vault")

Use of `abi.encodePacked()` used with dynamic types and passing the result to hash function such as `keccak256`, could result in hash collision.
This can be avoided by using `abi.encode()` since the items will be padded to 32 bytes.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L94						

## 2. NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES

	pragma solidity ^0.8.15;

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L4

This issue is found in all contract files.

## 3. `fees.deposit` SHOULD NOT BE GREATER THAN 100%. HENCE SHOULD BE CHECKED FOR `<=1e18` CONDITION.

In the `deposit()` function of the `Vault.sol` the fee for the deposit assets is calculated with the following line of code.

        uint256 feeShares = convertToShares(
            assets.mulDiv(uint256(fees.deposit), 1e18, Math.Rounding.Down)
        );
		
But here the fees.deposit is not checked for the `<=1e18` condition. During the initializtion of the `fees` the `<=1e18` is not checked. This could results in fees being charged greater than the deposit amount.
Hence following check should be added before the fee is calcualted. 

require(fees.deposit <= 1e18, "Fees can not be greater than 100%");

This should be checked for the fees.withdrawal, fees.management, fees.performance as well.

The fee updating function checks the above condtion even though it is not checked during the initialize function.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L143-L145
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L179

## 4. NEEDS TO CHECK `allowance(owner, msg.sender)` IS GREATER THAN THE `shares` BEING BURNED, AND SHOULD EMIT A REVERT MESSAGE IF IT IS NOT THE CASE.

In the `Vault.sol` contract inside the `withdraw` function before the required `shares` being burned, the `_approve` function is called to update the allowance for the `msg.sender` for the vault tokens owned by the `owner`.
But here the condition for the **allowance(owner, msg.sender) >= shares** is not checked.

        if (msg.sender != owner)
            _approve(owner, msg.sender, allowance(owner, msg.sender) - shares);
			
Instead revert is required with an error message if the **allowance(owner, msg.sender) < shares**.

Hence following line of code should be added to the code as follows:

        if (msg.sender != owner)
			require(allowance(owner, msg.sender) >= shares, "Not enough shares allowed for substraction")
            _approve(owner, msg.sender, allowance(owner, msg.sender) - shares);

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L230-L231
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L260-L261

## 5. `totalFee > 0` CONDITIONAL CHECK IS NOT REQUIRED, SINCE IT HAS ALREADY BEING DEEMED VALID BY A PREVIOUS CONDITION.

The `takeFees` modifier in the `Vault.sol` contract conducts the `totalFee > 0` check as shown in the following code snippet.

    modifier takeFees() {
        uint256 managementFee = accruedManagementFee();
        uint256 totalFee = managementFee + accruedPerformanceFee();
        uint256 currentAssets = totalAssets();
        uint256 shareValue = convertToAssets(1e18);

        if (shareValue > highWaterMark) highWaterMark = shareValue;

        if (managementFee > 0) feesUpdatedAt = block.timestamp;

        if (totalFee > 0 && currentAssets > 0)
            _mint(feeRecipient, convertToShares(totalFee));

        _;
    }

But there is no need to check `totalFee > 0`. Since if `managementFee > 0` then `totalFee` by default is > 0.
This is because as given in the code `uint256 totalFee = managementFee + accruedPerformanceFee()`;

Hence the `totalFee > 0` conditional check in the `if` statement can be omitted.

https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L480-L494