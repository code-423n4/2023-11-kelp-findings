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



