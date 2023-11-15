## The function `burn` in `src/interfaces/IRSETH.sol` not implement
In `src/interfaces/IRSETH.sol`
```
function burn(address account, uint256 amount) external;
```
while in `src/RSETH.sol`,there is only a similar function named `burnFrom`
```
    function burnFrom(address account, uint256 amount) external onlyRole(BURNER_ROLE) whenNotPaused {
        _burn(account, amount);
    }
```