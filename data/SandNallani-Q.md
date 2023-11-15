### [QA-1] Inconsistent use of the `whenNotPaused` modifier in the Oracle and Deposit pool contracts

The Oracle contract inherits and implements functions to pause and unpause. However, the `whenNotPaused` modifier is not applied to any of the functions in this contract. 

In the node delegator contract, when paused, cannot transfer assets back to the deposit pool. 
However, the deposit pool contract is able to transfer funds to the node delegator when paused.  


Consider removing the capability to pause/unpause the Oracle contract as it's not being used in the current version of the contract. 
Consider preventing the transfer of assets from the deposit pool to node delegator when paused, for better consistency. 

### [QA-2] The node delegator contract doesn’t track the number of shares created by the Eigen stragety.

The node delegator contract's responsible for depositing the user's deposit pool funds into various
Eigenlayer strategies. 
Shown below is the function that performs this task.
 
```
NodeDelegator::depositAssetIntoStrategy()
//NodeDelegator.sol
function depositAssetIntoStrategy(address asset){ 
...
emit AssetDepositIntoStrategy(asset, strategy, balance);

IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
...    
}
```
However, the number of shares minted by the Eigenlayer strategy aren't tracked or emitted as an argument in the event `AssetDepositIntoStrategy`. 
Consider checking the number of shares minted to ensuring that there's no adverse price impact and emitting it in the event to allow for off-chain accounting.




### [QA-3] The deposit pool's `depositAsset()` function could return the number of minted tokens

The main entry point in the Kelp protocol is the `LRTDepositPool::depositAsset()`. Which allows users to deposit approved assets into the pool to receive an amount of `rsETH` tokens. 
The minted `rsETH` tokens represent the user's underlying asset contribution of rETH, cbETH or stETH. 
If the user's a smart contract that intends to track the user's minted shares, it would be beneficial to return the value of the minted `rsETH` tokens. 

Consider returning the minting `rsETH` tokens by modifying the deposit function as follows.

```
    function getRsETHAmountToMint(
        address asset,
        uint256 amount
    )
        public
        view
        override
        returns (uint256 rsethAmountToMint) {
        ...
        rsethAmountToMint = (amount * lrtOracle.getAssetPrice(asset)) / lrtOracle.getRSETHPrice();
      }
```
