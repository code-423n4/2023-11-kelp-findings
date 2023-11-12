NodeDelegator.depositAssetIntoStrategy does not check if strategy address (lrtConfig.assetStrategy(asset)) is not address(0), since user when calling this function will transfer his funds via 

```
IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
```

which transfers from NodeDelegator to strategy on the given token address 


```
    function depositIntoStrategy(IStrategy strategy, IERC20 token, uint256 amount) external returns (uint256 shares) {
        token.transferFrom(msg.sender, address(strategy), amount);
```

Since the function does not check if strategy address (lrtConfig.assetStrategy(asset)) is not address(0), if `lrtConfig.assetStrategy(asset);` returns address(0) then NodeDelegator assets of `token` will be sent to address(0) (if token does not follows OZ pattern) and will be lost