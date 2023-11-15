In LRTOracle contract, for the `getAssetPrice` function. The `assetPriceOracle[asset] != 0` is being checked before calling that address with `getAssetPrice` function.
It would lead to unknown error, instead it would be better to revert at the start with custom error. 
