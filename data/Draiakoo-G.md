# Code optimizations
### First
Inside the [LRTConfig::initialize](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L41-L68) it is unnecessary to check for zero address for the stETH, rETH and cbETH because it is also checked inside the `_setToken` function.
These [lines of codes](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L52-L54) can be removed because in this [other function](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L157) is already checked for zero address.

### Second
Inside the [LRTDepositPool::transferAssetToNodeDelegator](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L183-L197) there is no need be nonReentrant, because it can only be used by a trusted role and there is no function in the protocol that triggers an external contract that can lead to reentrancy.

### Third
No need to make the LRTOracle pausable. This contract only provides with view functions, if the contracts that take his information is pausable is enough. It could be pausable for the function to change the price oracle `updatePriceOracleFor`, however it is not used.