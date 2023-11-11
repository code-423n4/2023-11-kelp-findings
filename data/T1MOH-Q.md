## 1. Low. `getAssetCurrentLimit()` will revert if admin sets new limit to less than deposited now
### Vulnerability Details
https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTDepositPool.sol#L56-L58

Limit from config can be lower than actually deposited, therefore function will revert instead of returning 0
```solidity
    function getAssetCurrentLimit(address asset) public view override returns (uint256) {
        return lrtConfig.depositLimitByAsset(asset) - getTotalAssetDeposits(asset);
    }
```

### Recommended mitigation steps
Return 0 instead of revert
```diff
    function getAssetCurrentLimit(address asset) public view override returns (uint256) {
-       return lrtConfig.depositLimitByAsset(asset) - getTotalAssetDeposits(asset);
+       return lrtConfig.depositLimitByAsset(asset) > getTotalAssetDeposits(asset)
+           ? lrtConfig.depositLimitByAsset(asset) - getTotalAssetDeposits(asset)
+           : 0;
    }
```

## 2. Low. Protocol assumes that all LSTs have 18 decimals
### Vulnerability Details
https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTOracle.sol#L52-L79

Protocol assumes that all LSTs have 18 decimals, it can be different in the future.
Here function expects that `totalAssetAmt` has 1e18 precission. Otherwise it will over/under-estimate asset:
```solidity
    function getRSETHPrice() external view returns (uint256 rsETHPrice) {
        address rsETHTokenAddress = lrtConfig.rsETH();
        uint256 rsEthSupply = IRSETH(rsETHTokenAddress).totalSupply();

        if (rsEthSupply == 0) {
            return 1 ether;
        }

        uint256 totalETHInPool;
        address lrtDepositPoolAddr = lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL);

        address[] memory supportedAssets = lrtConfig.getSupportedAssetList();
        uint256 supportedAssetCount = supportedAssets.length;

        for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
            address asset = supportedAssets[asset_idx];
            uint256 assetER = getAssetPrice(asset);

@>          uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
@>          totalETHInPool += totalAssetAmt * assetER;

            unchecked {
                ++asset_idx;
            }
        }

        return totalETHInPool / rsEthSupply;
    }
```

### Recommended mitigation steps
Normalize asset amount to 18 decimals:
```diff
        for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
            address asset = supportedAssets[asset_idx];
            uint256 assetER = getAssetPrice(asset);

            uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
+           uint256 decimals = IERC20(asset).decimals();
+           totalAssetAmt = decimals < 18 ? totalAssetAmt * (10 ** (18 - decimals)) : totalAssetAmt / (10 ** (decimals - 18));
            totalETHInPool += totalAssetAmt * assetER;

            unchecked {
                ++asset_idx;
            }
        }
```

## 3. Low. It is dangerous to give max approve to external contracts
### Vulnerability Details
https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/NodeDelegator.sol#L38

```solidity
    function maxApproveToEigenStrategyManager(address asset)
        external
        override
        onlySupportedAsset(asset)
        onlyLRTManager
    {
        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);
@>      IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);
    }
```

### Recommended mitigation steps
Consider giving approval before deposit:
```diff
    function depositAssetIntoStrategy(address asset)
        external
        override
        whenNotPaused
        nonReentrant
        onlySupportedAsset(asset)
        onlyLRTManager
    {
        address strategy = lrtConfig.assetStrategy(asset);
        IERC20 token = IERC20(asset);
        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

        uint256 balance = token.balanceOf(address(this));
+
+       token.approve(eigenlayerStrategyManagerAddress, balance);

        emit AssetDepositIntoStrategy(asset, strategy, balance);

        IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
    }
```