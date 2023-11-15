## LOW-RISK ISSUES

1. `addNodeDelegatorContractToQueue` is supposed to be called by LRTManager, but it is being called by LRTAdmin. `onlyLRTAdmin` is used, which is not the correct modifier. This can be seen from the comments below. Correct modifier needs to be used.

```
    /// @notice add new node delegator contract addresses
    /// @dev only callable by LRT manager
    /// @param nodeDelegatorContracts Array of NodeDelegator contract addresses
    function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {
        uint256 length = nodeDelegatorContracts.length;
        if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
            revert MaximumNodeDelegatorCountReached();
        }

        for (uint256 i; i < length;) {
            UtilLib.checkNonZeroAddress(nodeDelegatorContracts[i]);
            nodeDelegatorQueue.push(nodeDelegatorContracts[i]);
            emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]);
            unchecked {
                ++i;
            }
        }
    }
```

2. `transferAssetToNodeDelegator` does not have `whenNotpaused` modifier. A similar function, `depositAssetIntoStrategy` has, even though both functions are called by `onlyLRTManager`. Even, `transferBackToLRTDepositPool` also has the modifier. It is expected that when contracts are paused in an emergency, asset transfers should also be paused. But, that is not the case here.

```
    /// @notice transfers asset lying in this DepositPool to node delegator contract
    /// @dev only callable by LRT manager
    /// @param ndcIndex Index of NodeDelegator contract address in nodeDelegatorQueue
    /// @param asset Asset address
    /// @param amount Asset amount to transfer
    function transferAssetToNodeDelegator(
        uint256 ndcIndex,
        address asset,
        uint256 amount
    )
        external
        nonReentrant
        onlyLRTManager
        onlySupportedAsset(asset)
    {
        address nodeDelegator = nodeDelegatorQueue[ndcIndex];
        if (!IERC20(asset).transfer(nodeDelegator, amount)) {
            revert TokenTransferFailed();
        }
    }
```
