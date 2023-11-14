From the `LRTDepositPool` contract , the amount sent should be checked and revert early if not enough.
```
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
The following can be added to achieve the result
```
require(IERC20(asset).balanceOf(address(this) > 0),"error")
```