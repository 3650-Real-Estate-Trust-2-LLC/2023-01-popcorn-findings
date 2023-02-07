## LOW-1: State variable visibility not set

### Code

uint256 constant SECONDS_PER_YEAR = 365.25 days;
uint256 constant DEGRADATION_COEFFICIENT = 10**18;

### Affected file

* Vault.sol (Line: 35)
* YearnAdapter.sol (Line 25)

## LOW-2: Floating Pragma

### Affected file

* All files Line(4)