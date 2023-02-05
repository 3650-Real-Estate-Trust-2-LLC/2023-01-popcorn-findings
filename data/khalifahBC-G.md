## ( ++i )costs less gas than (i++), especially when it’s used in for-loops :-


contracts/PermissionRegistry.sol::42 => for (uint256 i = 0; i < len; i++) 
contracts/VaultController.sol::319 => for (uint8 i = 0; i < len; i++) 
contracts/VaultController.sol::337 => for (uint8 i = 0; i < len; i++) 
contracts/VaultController.sol::357 => for (uint8 i = 0; i < len; i++) 
contracts/VaultController.sol::374 => for (uint8 i = 0; i < len; i++) 
contracts/VaultController.sol::437 => for (uint256 i = 0; i < len; i++) 
contracts/VaultController.sol::494 => for (uint256 i = 0; i < len; i++) 
contracts/VaultController.sol::523 => for (uint256 i = 0; i < len; i++) 
contracts/VaultController.sol::564 => for (uint256 i = 0; i < len; i++) 
contracts/VaultController.sol::587 => for (uint256 i = 0; i < len; i++) 
contracts/VaultController.sol::607 => for (uint256 i = 0; i < len; i++) 
contracts/VaultController.sol::620 => for (uint256 i = 0; i < len; i++) 
contracts/VaultController.sol::633 => for (uint256 i = 0; i < len; i++) 
contracts/VaultController.sol::646 => for (uint256 i = 0; i < len; i++) 
contracts/VaultController.sol::766 => for (uint256 i = 0; i < len; i++) 
contracts/VaultController.sol::806 => for (uint256 i = 0; i < len; i++) 

##use ++i in stead of i++ for costing less gas
