# 1. Inconsistent Mapping Usage in CloneRegistry Contract

Link : https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L22

Summary: 

The `CloneRegistry` contract uses `mapping` data structures in an inconsistent manner, leading to potential security vulnerabilities.

Impact: 

The impact of this vulnerability is that the contract may not function as intended, leading to incorrect results when querying the mapping data structures. Additionally, if the data stored in the mappings are used to make decisions in other contracts, those contracts may also behave unexpectedly.

Recommendation: 

To mitigate this vulnerability, it is recommended to update the code to consistently use the mappings as intended. For example, it is better to initialize the values of the mappings in the constructor. This can be done by defining the default value for each `mapping` key or by setting the values for each key in the constructor.

Example: 

Here's an example of how the `mapping` can be initialized in the constructor:


```
mapping(address => bool) public cloneExists;
mapping(bytes32 => mapping(bytes32 => address[])) public clones;
address[] public allClones;

constructor(address _owner) Owned(_owner) {
    for (uint i = 0; i < allClones.length; i++) {
        allClones[i] = address(0);
    }
}
```

# 2. Lack of Input Validation in `addClone` function

Link : https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L41

Summary: 

The `addClone` function in the `CloneRegistry` contract does not perform any input validation, which can lead to incorrect data being stored on the blockchain.

Impact:

If an attacker is able to invoke the `addClone` function with malicious input, they may be able to add invalid clones to the registry. This can result in incorrect data being stored on the blockchain and potentially cause problems with other parts of the system that rely on the data stored in the registry.

Recommendation: 

Input validation should be added to the `addClone` function to ensure that only valid data can be stored in the registry. This can be done by checking the inputs for validity before storing them in the `mapping` and `arrays`. For example, the `templateCategory` and `templateId` inputs could be checked to ensure they are not empty or otherwise invalid.

Example:

```
  function addClone(
    bytes32 templateCategory,
    bytes32 templateId,
    address clone
  ) external onlyOwner {
    require(templateCategory != bytes32(0), "Invalid template category");
    require(templateId != bytes32(0), "Invalid template id");

    cloneExists[clone] = true;
    clones[templateCategory][templateId].push(clone);
    allClones.push(clone);

    emit CloneAdded(clone);
  }
```

# 3. Incorrect Access Control Vulnerability in `lock` Function

Link : https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L88

Summary: 

The `lock` function of the contract `MultiRewardEscrow` does not have proper access controls in place, allowing any user to lock tokens for any other user without their consent.

Impact: 

This vulnerability can result in an unauthorized user locking the funds of another user, leading to a potential loss of funds.

Recommendation: 

The access control mechanism should be implemented to ensure that only authorized users are able to lock tokens for others. This can be done by implementing an access control mechanism that restricts the call to the `lock` function based on the address of the token owner.

Example:

```
function lock(
    IERC20 token,
    address account,
    uint256 amount,
    uint32 duration,
    uint32 offset
  ) external {
    require(msg.sender == token.owner(), "Unauthorized user");

    if (token == IERC20(address(0))) revert ZeroAddress();
    if (account == address(0)) revert ZeroAddress();
    if (amount == 0) revert ZeroAmount();
    if (duration == 0) revert ZeroAmount();

    ...
  }
```

# 4. Unrestricted access to escrowed tokens

Link : https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L51

Summary: 

The `MultiRewardEscrow` contract allows anyone to retrieve information about all existing escrows, including their token type, account address, and amount, by calling the `getEscrows()` function.

Impact: 

An attacker can exploit this vulnerability to retrieve sensitive information about the escrowed tokens, which could be used to carry out further attacks or malicious activities.

Recommendation: 

To prevent the unauthorized retrieval of escrow information, the `getEscrows()` function should be restricted to only return the information of escrows owned by the caller. This can be done by adding an access control mechanism that checks the caller's address against the account addresses associated with each escrow.

Example:

```
function getEscrows(bytes32[] calldata escrowIds) external view returns (Escrow[] memory) {
  Escrow[] memory selectedEscrows = new Escrow[](escrowIds.length);
  for (uint256 i = 0; i < escrowIds.length; i++) {
    if (msg.sender == escrows[escrowIds[i]].account) {
      selectedEscrows[i] = escrows[escrowIds[i]];
    }
  }
  return selectedEscrows;
}
```

# 5. Improper Validation of `Deposit` Function

Link: https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L107

Summary: 

The `_deposit` function in the `MultiRewardStaking` contract does not properly validate the inputs, potentially leading to security issues.

Impact: 

If the inputs are not properly validated, attackers may be able to exploit the contract and cause harm. For example, they may be able to deposit an incorrect amount or deposit to an incorrect address. This could lead to a loss of funds or incorrect distribution of rewards.

Recommendation: 

It is recommended that proper input validation be added to the `_deposit` function to ensure the inputs are correct and secure. For example, the input `amount` should be checked to ensure it is not `0` and the `receiver` address should be checked to ensure it is not the null address.

Example:

```
function _deposit(
    address caller,
    address receiver,
    uint256 assets,
    uint256 shares
  ) internal override accrueRewards(caller, receiver) {
    require(assets > 0, "Deposit amount must be greater than 0");
    require(receiver != address(0), "Receiver address cannot be null");
    
    IERC20(asset()).safeTransferFrom(caller, address(this), assets);

    _mint(receiver, shares);

    emit Deposit(caller, receiver, assets, shares);
  }
```
