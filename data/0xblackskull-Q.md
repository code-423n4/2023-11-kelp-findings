LRTOracle:getRSETHPrice
- Use `uint256 asset_idx` instead of `uint16 asset_idx`  in for loop because `supportedAssetCount` is in `uint256` format
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L66

LRTDepositPool:addNodeDelegatorContractToQueue
- Duplicate array check missing in addNodeDelegatorContractToQueue
- Don't use `=>` instead of `=`, as it is wrong in bot-report[https://github.com/code-423n4/2023-11-kelp/blob/main/bot-report.md#g-14--costs-less-gas-than-] otherwise it will revert on 10 
  https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol

LRTDepositPool:getRsETHAmountToMint
- `onlySupportedAsset(asset)` modifier missing in getRsETHAmountToMint
  https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L162