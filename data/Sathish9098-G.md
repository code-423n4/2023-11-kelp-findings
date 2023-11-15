# GAS OPTIMIZATIONS

##

## [G-1] Consolidating Multiple Address/ID Mappings into a Single Struct-Based Mapping for Efficiency

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

By consolidating the mappings into a single struct-based mapping, potentially save ``1 SLOT`` and ``2000 GAS``  per token address. This reduction can lead to significant gas savings, especially in contracts with many entries.

```diff
FILE: 2023-11-kelp/src/LRTConfig.sol

- 15: mapping(address token => bool isSupported) public isSupportedAsset;
- 16: mapping(address token => uint256 amount) public depositLimitByAsset;
- 17: mapping(address token => address strategy) public override assetStrategy;

+ struct AssetInfo {
+    bool isSupported;
+    address strategy;
+    uint256 depositLimit;
+ }

+ mapping(address => AssetInfo) public assetInfo;

```
https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L13-L14

##

## [G-2] Don't cache state variables or function calls only used once 

```diff
FILE: Breadcrumbs2023-11-kelp/src/LRTDepositPool.sol

function _mintRsETH(address _asset, uint256 _amount) private returns (uint256 rsethAmountToMint) {
        (rsethAmountToMint) = getRsETHAmountToMint(_asset, _amount);

-        address rsethToken = lrtConfig.rsETH();
        // mint rseth for user
-        IRSETH(rsethToken).mint(msg.sender, rsethAmountToMint);
+        IRSETH(lrtConfig.rsETH()).mint(msg.sender, rsethAmountToMint);
    }



- 193: address nodeDelegator = nodeDelegatorQueue[ndcIndex];
-        if (!IERC20(asset).transfer(nodeDelegator, amount)) {
+        if (!IERC20(asset).transfer(nodeDelegatorQueue[ndcIndex], amount)) {
            revert TokenTransferFailed();
        }

```
https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L154

##

## [G-3] Don't declare variables inside the loop

Declare outside the loop and only use inside

```diff
FILE: Breadcrumbs2023-11-kelp/src/LRTOracle.sol

ddress[] memory supportedAssets = lrtConfig.getSupportedAssetList();
        uint256 supportedAssetCount = supportedAssets.length;
+    address asset;
+    uint256 assetER;
+ uint256 totalAssetAmt;
        for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
-            address asset = supportedAssets[asset_idx];
+             asset = supportedAssets[asset_idx];
-            uint256 assetER = getAssetPrice(asset);
+            assetER = getAssetPrice(asset);

-            uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
+            totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
            totalETHInPool += totalAssetAmt * assetER;

            unchecked {
                ++asset_idx;
            }
        }

        return totalETHInPool / rsEthSupply;
    }

```






