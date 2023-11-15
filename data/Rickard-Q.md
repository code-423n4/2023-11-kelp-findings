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
For functions such as [checkNonZeroAddress](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/UtilLib.sol#L11), it is more readable to have this function be named `_checkNonZeroAddress` so readers know it is an internal function. A similarly opinionated recommendation is to use `s_` for storage variables and `i_` for immutable variables.
````solidity
11:    function checkNonZeroAddress(address address_) internal pure {
12:        if (address_ == address(0)) revert ZeroAddressNotAllowed();
13:    }
````
[https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/UtilLib.sol#L11-L13](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/UtilLib.sol#L11-L13)