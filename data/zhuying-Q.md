# Lines of code
https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L78
https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L109

# Summary
Precision loss in ```getRsETHAmountToMint```

# Vulnerability Detail
## Impact
Precision loss in ```getRsETHAmountToMint```. Honestly the impact is very low.

## Vulnerability Detail
```
 rsethAmountToMint = (amount * lrtOracle.getAssetPrice(asset)) / lrtOracle.getRSETHPrice();
```
According to ```LRTOracle.sol```, RSETHPrice = totalETHInPool / rsEthSupply.So it will cause precision loss because of including div before mul.

## Tool Used
Manual

## Recommendation
The calculation can change to:
```
rsethAmountToMint = amount * lrtOracle.getAssetPrice(asset) * rsEthSupply / totalETHInPool;
```

