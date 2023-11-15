# [L-01] Modifier `onlySupportedAsset()` is missing in `getAssetBalance()`

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L121-L124

## Description
The method `getAssetBalance()` doesn't check if the asset is a supportedAsset or not. Which can revert or will return wrong values if **assetStrategy** is present for that asset.

## Recommendations
Add `onlySupportedAsset()` modifier in `getAssetBalance()` method
```
-    function getAssetBalance(address asset) external view override returns (uint256) {
+    function getAssetBalance(address asset) external view override onlySupportedAsset(asset) returns (uint256) {
         address strategy = lrtConfig.assetStrategy(asset);
         return IStrategy(strategy).userUnderlyingView(address(this));
     }
```
# [L-02] While depositing assets in the LRTDepositPool, method getAssetPrice() can revert unexpectedly if price feed for the asset is not present in assetPriceFeed array

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L45-L47

## Summary
The `getAssetPrice()` returns to price of an asset by fetching in from Chainlink oracle. It checks `onlySupportedAsset()` modifier but doesn't check if the price feed for this asset is present or not.

## Vulnerability Details
The `getAssetPrice()` doesn't check if the price feed for the asset is added. It can revert unexpectedly when calculating RSETH or asset prices.

```
function getAssetPrice(address asset) external view onlySupportedAsset(asset) returns (uint256) {
        return AggregatorInterface(assetPriceFeed[asset]).latestAnswer();
    }
```

# [L-03] **rsethAmountToMint > 0** check is missing when executing `_mintRsETH()`

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L151-L157

## Description
The method `_mintRsETH()` mint and sends RSETH to the caller. But the check **rsethAmountToMint != 0** is missing in the function. 

## Recommendations
```
if(rsethAmountToMint == 0) {
  revert("RSETH amount is 0");
}
```

### POC Test:
Add the following test in ChainlinkPriceOracleTest.t.sol
```
contract ChainlinkPriceOracleFetchAssetPriceMissingPriceFeed is ChainlinkPriceOracleTest {
    MockPriceAggregator public priceFeed;

    function setUp() public override {
        super.setUp();
        priceOracle.initialize(address(lrtConfig));
        priceFeed = new MockPriceAggregator();

        vm.prank(manager);
    }

    function test_FetchAssetPriceWithMissingPriceFeed() external {
        vm.expectRevert();
        priceOracle.getAssetPrice(address(rETH));
    }
}
```

## Recommendations
Handle missing price feed expection gracefully
```
     function getAssetPrice(address asset) external view onlySupportedAsset(asset) returns (uint256) {
-        return AggregatorInterface(assetPriceFeed[asset]).latestAnswer();
+        address priceFeed = assetPriceFeed[asset];
+        if(priceFeed == address(0)) {
+            revert("Missing price feed");
+        }
+        return AggregatorInterface(priceFeed).latestAnswer();
     }
```

# [NC-01] There is a mistake in comments in `LRTConfig` contract

## Description
Update the comment as below:
```
/// @param cbETH cbETH address
-    /// @param rsETH_ cbETH address
+    /// @param rsETH_ rsETH_ address
```

# [NL-02] Public variable **supportedAssetList** can be set as private as a getter is present in the contract

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L19
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L135-L137

## Description
Pubic variable **supportedAssetList** can be set as private. Because there is a getter present in the contract.

## Recommendations
```
address[] private supportedAssetList;
```
