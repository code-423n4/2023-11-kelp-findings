### [L-01] Supported Assets cannot be removed in LRTConfig.sol

The asset and deposit limit can be added to the `isSupportedAsset` mapping and is pushed to the `supportedAssetList` address array, but cannot be popped nor set to false. It would be great if the asset can be popped from the array if the supported token is not in use, or simply set to false. A way to make sure that the supported asset is not in use is to set the deposit limit to zero.

```
    /// @dev Adds a new supported asset
    /// @param asset Asset address
    /// @param depositLimit Deposit limit for the asset
    function _addNewSupportedAsset(address asset, uint256 depositLimit) private {
        UtilLib.checkNonZeroAddress(asset);
        if (isSupportedAsset[asset]) {
            revert AssetAlreadySupported();
        }
        isSupportedAsset[asset] = true;
        supportedAssetList.push(asset);
        depositLimitByAsset[asset] = depositLimit;
        emit AddedNewSupportedAsset(asset, depositLimit);
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol

### [L-02] NodeDelegator cannot be removed from the nodeDelegatorQueue array

There is a `maxNodeDelegatorCount` of 10 set in the initializer, and this number can be changed. Node delegators are added to the `nodeDelegatorQueue`, but they cannot be removed. Consider allowing the removal of node delegators. This is quite important as the `nodeDelegatorQueue` is used in a loop, and if every node delegator has some assets present, the `getAssetDistributionData()` might run into a out of gas error when calculating the amount of assets in the NDCs

```
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

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L71

### [L-03] No burn function exposed in rsETH.sol

Users cannot withdraw their LST once they minted the LRT. Consider allowing the burning of rsETH to get LST back.

```
    // No burn function, only mint
    function mint(address to, uint256 amount) external onlyRole(MINTER_ROLE) whenNotPaused {
        _mint(to, amount);
    }
```