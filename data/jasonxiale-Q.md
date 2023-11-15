# [L-01] LRTDepositPool.sol and LRTOracle.sol are inherited from PausableUpgradeable, but there are no functions using paused related modifier
File:
https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L19
https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L19
Impact:
LRTDepositPool.sol and LRTOracle.sol are inherited from PausableUpgradeable, but there are no functions using paused related modifier

# [L-02] `LRTDepositPool.getRsETHAmountToMint` lacks `onlySupportedAsset` modifier as `LRTDepositPool.getAssetDistributionData`
File:
https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L71-L76
https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L95-L102
Function `LRTDepositPool.getAssetDistributionData` has `onlySupportedAsset` modifier, but `LRTDepositPool.getRsETHAmountToMint` doesn't have, I think it's better to add one.


# [L-03] `LRTDepositPool.sol` lacks of function to remove Node Delegator
File:
    https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol
In contract `LRTDepositPool.sol`, there is a function [LRTDepositPool.addNodeDelegatorContractToQueue](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L162-L176) to add new Node to `LRTDepositPool.nodeDelegatorQueue`, I think it's better to add a function to do the reverse operation - remove Node Delegator