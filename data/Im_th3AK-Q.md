# Risk Rating 
- Low 
# Description
- missing zero address validation.
LRTConfig.setRSETH(address).rsETH_ (src/LRTConfig.sol#144) lacks a zero-check on :
- rsETH = rsETH_ (src/LRTConfig.sol#146)
# Impact
Link:- https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L144-L146
```
 function setRSETH(address rsETH_) external onlyRole(DEFAULT_ADMIN_ROLE) {
        UtilLib.checkNonZeroAddress(rsETH_);
        rsETH = rsETH_;
    }
```
It lacks a zero-check on the rsETH_ parameter, which is an address type. This means that the function setRSETH could potentially accept the zero address (0x0000000000000000000000000000000000000000) as a valid input and assign it to the rsETH state variable. This could have serious consequences, such as:

- The zero address could be used to bypass some access control or validation logic that relies on the rsETH variable.
- The zero address could be used to lock funds or tokens that are sent to the rsETH variable, since the zero address is not owned by anyone and cannot be controlled.
- The zero address could be used to cause unexpected behavior or errors in other functions that interact with the rsETH variable.
# Tools 
- Manual
# Recommendations
```
function setRSETH(address rsETH_) external onlyRole(DEFAULT_ADMIN_ROLE) {
    // Check that rsETH_ is not the zero address
    require(!iszero(rsETH_), "rsETH_ cannot be the zero address");
    UtilLib.checkNonZeroAddress(rsETH_);
    rsETH = rsETH_;
}

```
This code ensures that the function setRSETH will revert if rsETH_ is the zero address, and prevents any potential security or functionality risks.