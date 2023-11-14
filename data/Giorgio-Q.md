## Misleading comment in `LTRDepositPool::addNodeDelegatorContractToQueue()`

## Issue

The comment above the `addNodeDelegatorContractToQueue()` function indicate that the `manager` should call this function, but the modifier used is for `admin`. This is confusing.

## Link

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L160-L162

## Fix

To fix this we should change the comments accordingly so that they can be in line with the code.

```
 /// @dev only callable by LRT admin
```
