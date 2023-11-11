## [N‑01] All `override` functions in LRTDepositPool contract, NodeDelegator contract, LRTConfig contract and LRTOracle contract should have the keyword "override" removed.

Many functions in this protocol use the keyword `override`. Nevertheless, these functions don't override anything and are just implementations of the inherited interface.

Therefore, the following functions should have the useless keyword `override` removed:

LRTDepositPool contract (view functions):
- `getTotalAssetDeposits` function
- `getAssetCurrentLimit` function
- `getNodeDelegatorQueue` function
- `getAssetDistributionData` function
- `getRsETHAmountToMint` function

NodeDelegator contract (view and write functions):
- `maxApproveToEigenStrategyManager` function
- `depositAssetIntoStrategy` function
- `getAssetBalances` function
- `getAssetBalance` function

LRTConfig contract (view functions):
- `getLSTToken` function
- `getContract` function
- `getSupportedAssetList` function
- `assetStrategy` public mapping (getter function automatically generated)

LRTOracle contract:
- `assetPriceOracle` public mapping (getter function automatically generated)

This will make the code easier to read. Indeed, currently, one might look for the corresponding `virtual function` for all `override` functions, though they do not exist.


## [N‑02]


