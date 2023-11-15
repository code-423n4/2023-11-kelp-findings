# 1. Potential Data Inconsistency between `tokenMap` and `isSupportedAsset`

In contract `LRTConfig`, the token info is split into two storage: 

- ``tokenMap`` 
-  `isSupportedAsset`.

And each storage will be updated by the different role:

- ``tokenMap`` -> admin
-  `isSupportedAsset` -> manager

As a result, there could be data inconsistency between the two storages.



# 2. Unused Pause Functionality

The `LRTOracle` declare pause and unpause but no function is restricted with these methods.



# 3. Missing Truncation Protection in `getRsETHAmountToMint`

`getRsETHAmountToMint` will return the value that user can mint, however, there could be truncation when rsETH price is large or amount is too low. As a result user will deposit for nothing:

```
rsethAmountToMint = (amount * lrtOracle.getAssetPrice(asset)) / lrtOracle.getRSETHPrice();
```

It's better to set a minimum deposit value(amount * asset value).