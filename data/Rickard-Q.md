## [L-01] The `@dev` NatSpec comment in `UtilLib.checkNonZeroAddress` is incorrect
The NatSpec for the `checkNonZeroAddress()` function should indicate that it is a function, not a modifier.
````diff
src/utils/UtilLib.sol
-   /// @dev zero address check modifier
+   /// @dev zero address check function
    /// @param address_ address to check
    function checkNonZeroAddress(address address_) internal pure {
        if (address_ == address(0)) revert ZeroAddressNotAllowed();
    }
````
## [L-02] Use underscore prefix for internal functions
## Relevant GitHub Links

## Summary

## Vulnerability Details
````solidity

````
## Impact

## Tools Used

## Recommendations