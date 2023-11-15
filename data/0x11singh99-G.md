## Gas Optimizations

| Number                                                                                                 | Issue                                                                                   | Instances |
| ------------------------------------------------------------------------------------------------------ | :-------------------------------------------------------------------------------------- | :-------: |
| [[G-01](#g-01-multiple-mappings-that-share-an-id-can-be-combined-into-a-single-mapping-of-id--struct)] | Multiple mappings that share an ID can be combined into a single mapping of ID / struct |     3     |
| [[G-02](#g-02-can-make-the-variable-outside-the-loop-to-save-gas)]                                     | Can Make The Variable Outside The Loop To Save Gas                                      |     3     |
| [[G-03](#g-03-do-not-cache-function-call-if-used-only-once)]                                           | Do not cache function call if used only once                                            |     1     |

## [G-01] Multiple mappings that share an ID can be combined into a single mapping of ID / struct

This can avoid a Gsset (20000 Gas) per mapping combined. Reads and writes will also be cheaper when a function requires both values as they both can fit in the same storage slot.

Finally, if both fields are accessed in the same function, this can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 Gas) and that calculation's associated stack operations.

**3 Instances**

Since all these 3 mappings shares same key so their value can be combined into one struct that is mapped by that same key. Even further those struct variables can be packed ie. `address strategy` will be packed with `bool isSupported` into one slot and it can save 1 SLOT whenever new entry comes into newly updated mapping.

```solidity
File : src/LRTConfig.sol

15:    mapping(address token => bool isSupported) public isSupportedAsset;
16:    mapping(address token => uint256 amount) public depositLimitByAsset;
17:    mapping(address token => address strategy) public override assetStrategy;

```

[15-17](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L15C4-L17C78)

```diff
File : src/LRTConfig.sol

contract LRTConfig is ILRTConfig, AccessControlUpgradeable {

+    struct AssetData {
+        uint256 depositLimit;
+        address strategy;
+        bool isSupported;  //@audit packed this bool strategy slot. 1 SLOT saved.
+    }

+   mapping(address => AssetData) public assetDetails;

    mapping(bytes32 tokenKey => address tokenAddress) public tokenMap;
    mapping(bytes32 contractKey => address contractAddress) public contractMap;
-   mapping(address token => bool isSupported) public isSupportedAsset;
-   mapping(address token => uint256 amount) public depositLimitByAsset;
-   mapping(address token => address strategy) public override assetStrategy;


```

## [G-02] Can Make The Variable Outside The Loop To Save Gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements. By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract

**3 Instances**

```solidity
File : src/LRTOracle.sol

66:        for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
67:            address asset = supportedAssets[asset_idx];
68:            uint256 assetER = getAssetPrice(asset);
69:
70:            uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
71:            totalETHInPool += totalAssetAmt * assetER;
72:
73:            unchecked {
74:                ++asset_idx;
75:            }
76:        }

```

[66-76](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L66C1-L76C10)

```diff
File : src/LRTOracle.sol

+      address asset;
+      uint256 assetER;
+      uint256 totalAssetAmt;

        for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
-           address asset = supportedAssets[asset_idx];
-           uint256 assetER = getAssetPrice(asset);

-           uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
+            asset = supportedAssets[asset_idx];
+            assetER = getAssetPrice(asset);

+            totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
            totalETHInPool += totalAssetAmt * assetER;

            unchecked {
                ++asset_idx;
            }
        }

```

## [G-03] Do not cache function call if used only once

### No need to cache `getAssetPrice(asset)` because it is used only once

**1 Instance**

```solidity
File : src/LRTOracle.sol

66:        for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
67:            address asset = supportedAssets[asset_idx];
68:            uint256 assetER = getAssetPrice(asset);
69:
70:            uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
71:            totalETHInPool += totalAssetAmt * assetER;
72:
73:            unchecked {
74:                ++asset_idx;
75:            }
76:        }

```

[66-76](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L66C1-L76C10)

```diff

        for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
            address asset = supportedAssets[asset_idx];
-           uint256 assetER = getAssetPrice(asset);

            uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
-           totalETHInPool += totalAssetAmt * assetER;
+           totalETHInPool += totalAssetAmt * getAssetPrice(asset);

            unchecked {
                ++asset_idx;
            }
        }
```
