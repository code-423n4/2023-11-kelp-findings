## [L-1] `addNodeDelegatorContractToQueue` doesn't check nodeDelegatorContracts duplicate
https://github.com/code-423n4/2023-11-kelp/blob/ee1154fcb6f6619cdc9aeda27503d9a2cbf6d8eb/src/LRTDepositPool.sol#L168-L175

 It's possible to add  duplicate nodes to queue, and there is no function to delete from  `nodeDelegatorQueue`.  Once duplicate node is added, duplicate node data will sum twice,this will affect the `deposit` logic and price calculation.


## [L-2] `getAssetBalance` should check asset is valid 
https://github.com/code-423n4/2023-11-kelp/blob/ee1154fcb6f6619cdc9aeda27503d9a2cbf6d8eb/src/NodeDelegator.sol#L121
should check  the input asset is valid or not 

## [L-3]  `pause` never gets used  in this `NodeDelegator`
https://github.com/code-423n4/2023-11-kelp/blob/ee1154fcb6f6619cdc9aeda27503d9a2cbf6d8eb/src/NodeDelegator.sol#L127-L134

`pause` is defined, but never get used.

## [L-4] `updateMaxNodeDelegatorCount`, new `maxNodeDelegatorCount_` should higher than current node length
https://github.com/code-423n4/2023-11-kelp/blob/ee1154fcb6f6619cdc9aeda27503d9a2cbf6d8eb/src/LRTDepositPool.sol#L202-L205

but there is no such check 

