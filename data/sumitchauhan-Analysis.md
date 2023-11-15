Low risk issue-
1. Calls inside a loop
LRTConfigRoleChecker.onlySupportedAsset(address) has external calls inside a loop: ! lrtConfig.isSupportedAsset(asset) 
LRTOracle.getAssetPrice(address) has external calls inside a loop: IPriceFetcher(assetPriceOracle[asset]).getAssetPrice(asset) 
LRTOracle.getRSETHPrice() has external calls inside a loop: totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset) 
NodeDelegator.getAssetBalances() has external calls inside a loop: assets[i] = address(IStrategy(strategies[i]).underlyingToken()) 
NodeDelegator.getAssetBalances() has external calls inside a loop: assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this))
 
Gas Optimizations-
1. Multiple mappings can be replaced with a single struct mapping:
  LRTConfig.sol::85 => isSupportedAsset[asset] = true;
  LRTConfig.sol::87 => depositLimitByAsset[asset] = depositLimit;
  LRTConfig.sol::102 => depositLimitByAsset[asset] = depositLimit;
  LRTConfig.sol::118 => if (assetStrategy[asset] == strategy) {
  LRTConfig.sol::121 => assetStrategy[asset] = strategy;
  LRTConfig.sol::158 => if (tokenMap[key] == val) {
  LRTConfig.sol::161 => tokenMap[key] = val;
  LRTConfig.sol::174 => if (contractMap[key] == val) {
  LRTConfig.sol::177 => contractMap[key] = val;
  ChainlinkPriceOracle.sol::47 => assetPriceFeed[asset] = priceFeed;
  LRTOracle.sol::97 => assetPriceOracle[asset] = priceOracle;
  
2. Constants in comparisons should appear on the left side:
  LRTDepositPool.sol::129 => if (depositAmount == 0) {
  
3. `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables:
  LRTOracle.sol::71 => totalETHInPool += totalAssetAmt * assetER;
  LRTOracle.sol::56 => if (rsEthSupply == 0) {
  LRTDepositPool.sol::83 => assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
  LRTDepositPool.sol::84 => assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
  

### Time spent:
8 hours