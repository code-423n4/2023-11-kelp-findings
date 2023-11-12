The `NodeDelegator` contract and the `RSETH` contract allows for pausing and unpausing via the `pause ()` and `unpause` function. The contract code on the `pause ()` function reads as `Only callable by LRT config manager. Contract must NOT be paused.` and `unpause ()` reads as `Only callable by the rsETH admin. Contract must be paused` 

Clearly, the intent of the project is to ensure the `pause ()` function is called only when the contract is NOT paused and vice versa for `unpause ()`. 

To ensure this is the case, the project should consider adding the `whenPaused` and `whenNotPaused` modifier on the two functions as follows:

```
 function pause() external onlyLRTManager whenNotPaused {
        _pause();
    }

    /// @notice Returns to normal state.
    /// @dev Only callable by the rsETH admin. Contract must be paused
    function unpause() external onlyRole(DEFAULT_ADMIN_ROLE) whenPaused {
        _unpause();
    }
```