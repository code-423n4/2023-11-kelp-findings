### GAS 01 - NodeDelegator.sol:110 -> cache **IStrategy(strategies[i])**: (Saves 118 gas)

```solidity
110: assets[i] = address(IStrategy(strategies[i]).underlyingToken());
111: assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
```
**Suggested edit:**

```solidity
IStrategy strategy = IStrategy(strategies[i]);
assets[i] = address(strategy.underlyingToken());
assetBalances[i] = strategy.userUnderlyingView(address(this));
```

| Function Name                                | min             | avg   | median | max    | # calls |
|----------------------------------------------|-----------------|-------|--------|--------|---------|
| getAssetBalances        (Before)             | 24953           | 24953 | 24953  | 24953  | 1       |
| getAssetBalances        (After)              | 24835           | 24835 | 24835  | 24835  | 1       |

### GAS 02 - LRTDepositPool.sol:83 -> cache **nodeDelegatorQueue[i]** as a variable (Saves 837 gas)

```solidity
83: assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
84: assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
```

**Suggested edit:**
```solidity
address currentNdc = nodeDelegatorQueue[i];
assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
```


| Function Name                                  | min             | avg   | median | max    | # calls |
|------------------------------------------------|-----------------|-------|--------|--------|---------|
| getAssetDistributionData     (Before)          | 10153           | 10153 | 10153  | 10153  | 1       |
| getAssetDistributionData     (After)           | 9316            | 9316  | 9316   | 9316   | 1       |

### GAS 03 - LRTOracle.sol:68 -> directly use **getAssetPrice(asset)** and **ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset)** instead of declaring variables **(assetER, totalAssetAmt)** which are used once -> saves 48 gas on average

```solidity
67: address asset = supportedAssets[asset_idx];
68: uint256 assetER = getAssetPrice(asset);

70: uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
71: totalETHInPool += totalAssetAmt * assetER;
```

**Suggested edit:**
```solidity
address asset = supportedAssets[asset_idx];
            
totalETHInPool += ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset) * getAssetPrice(asset);
```

| Function Name                        | min             | avg   | median | max   | # calls |
|--------------------------------------|-----------------|-------|--------|-------|---------|
| getRSETHPrice          (Before)      | 15460           | 36558 | 36558  | 57656 | 2       |
| getRSETHPrice          (After)       | 15460           | 36510 | 36510  | 57560 | 2       |
