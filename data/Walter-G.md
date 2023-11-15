## NodeDelegator
1) IERC20(token) in line 60(https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L60C1-L60C1) could be implemented directly inside the lines without creating temp variable,following the standard of other functions like maxApproveToEigenStrategyManager,in this way every call of the depositAssetIntoStrategy will save 2 gas
```
    function depositAssetIntoStrategy(address asset)
        external
        override
        whenNotPaused
        nonReentrant
        onlySupportedAsset(asset)
        onlyLRTManager
    {
        address strategy = lrtConfig.assetStrategy(asset);
        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

        uint256 balance = IERC20(asset).balanceOf(address(this));

        emit AssetDepositIntoStrategy(asset, strategy, balance);

        IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), IERC20(asset), balance);
    }
```