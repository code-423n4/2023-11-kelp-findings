### No Address check

InLRTDepositPool.sol/transferAssetToNodeDelegator(), there was no address check on address nodeDelegator. If a zero address is filled in by mistake that will to a serious loss of funds

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L183C3-L197C6