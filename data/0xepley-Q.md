## 1): Wrong event emittion
In the function `depositAssetIntoStrategy` of `NodeDelegator` the event
```solidity
emit AssetDepositIntoStrategy(asset, strategy, balance);
```

is being emitted but the location of this event emission is wrong, the event is emitted before the tokens are being deposited into the `strategy` so make sure to correct it and emit the event after the assets are being deposited into the `strategy`

## 2): Approving max to Eigen Layer
The function `maxApproveToEigenStrategyManager` is approving max (`uint256`) to eigen layer strategy which is really dangerous if the strategy is malicious so it is better to approve it according to the needs

## 3): `nodeDelegatorQueue` can be DOS
The smart contract is storing all the node Delegators into an array, although the initial limit is 10 delegators but it can be increase so an `uin256.max` amount and storing such an amount in an array is dangerous as the code is alliterating over the array and the array can be DOS.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L202-L205

## 4): NO check on amount being transfered
In the function `transferAssetToNodeDelegator` `onlyLRTManager` is transfering the money deposited into the node Delegator but there isn't any check on the amount he passed, it could be zero or could be more then what the contract is currently holding, so it is recommended to add a check for this purpose

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L183-L197