### [G-01] Cache multiple accesses of a mapping/array

Consider using a local `storage` or `calldata` variable when accessing a mapping/array value multiple times.

This can be useful to avoid recalculating the mapping hash and/or the array offsets.

- https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L109

```solidity
File: src/NodeDelegator.sol
109: for (uint256 i = 0; i < strategiesLength;) {
            assets[i] = address(IStrategy(strategies[i]).underlyingToken()); //@audit cache variable
            assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
            unchecked {
                ++i;
            }
        }
```

### [G-02] Move the declaration of user outside the loop to reduce SLOAD (storage load) operations, as storage reads can be expensive in terms of gas.

Optimize gas usage by avoiding variable declarations inside loops

- https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L66

```solidity
File: src/LRTOracle.sol
66: for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
            address asset = supportedAssets[asset_idx]; //@audit make variable outside loop
            uint256 assetER = getAssetPrice(asset);

            uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
            totalETHInPool += totalAssetAmt * assetER;

            unchecked {
                ++asset_idx;
            }
        }
```

```diff
File: src/LRTOracle.sol
+             address asset;
+             uint256 assetER;
66: for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
-            address asset = supportedAssets[asset_idx];
-            uint256 assetER = getAssetPrice(asset);
+             asset = supportedAssets[asset_idx];
+             assetER = getAssetPrice(asset);

            uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
            totalETHInPool += totalAssetAmt * assetER;

            unchecked {
                ++asset_idx;
            }
        }
```

### [G-03] Use assembly for back to back external call

- https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L61

```solidity
File: src/NodeDelegator.sol
61: address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

63: uint256 balance = token.balanceOf(address(this));
```

- https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L61

```solidity
File: src/LRTOracle.sol
61: address lrtDepositPoolAddr = lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL);

63: address[] memory supportedAssets = lrtConfig.getSupportedAssetList();
```

### [G-04] Use stack variable in emit instead of state variable

- https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/LRTOracle.sol#L66

```solidity
File: src/LRTOracle.sol
202: function updateMaxNodeDelegatorCount(uint256 maxNodeDelegatorCount_) external onlyLRTAdmin {
        maxNodeDelegatorCount = maxNodeDelegatorCount_;
        emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount);
    }
```

```diff
File: src/LRTOracle.sol
202: function updateMaxNodeDelegatorCount(uint256 maxNodeDelegatorCount_) external onlyLRTAdmin {
        maxNodeDelegatorCount = maxNodeDelegatorCount_;
-        emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount);
+        emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount_);
    }
```