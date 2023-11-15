### Summary

* \[L-01\] `LRTConfig::updateAssetDepositLimit()` function doesn't check if the current deposits are greater than the new limit
    
* \[L-02\] `LRTDepositPool::addNodeDelegatorContractToQueue()` function should check if the inputted array includes the same addresses
    
* \[L-03\] `LRTDepositPool::updateMaxNodeDelegatorCount` should check the new count is not below the current delegator count
    
* \[L-04\] Deposited amounts in the EigenLayer strategy should be checked before updating the strategy for the asset
    
* \[L-05\] `ChainlinkPriceOracle::getAssetPrice()` function should check the stale price
    
* \[NC-01\] `ChainlinkPriceOracle.sol` contract should use AggregatorV3Interface instead of AggregatorInterface
    
* \[NC-02\] It would be better to check the decimal precision of the returned values from oracles
    

---

### \[L-01\] `LRTConfig::updateAssetDepositLimit()` function doesn't check if the current deposits are greater than the new limit

[https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L94C5-L104C6](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L94C5-L104C6)

Every asset has a deposit limit and the contract managers can update these limits. The issue here is that the `updateAssetDepositLimit()` function doesn't check the current total asset deposits.

It is possible to set a new limit below the current deposits which leads to a broken invariant.

**Recommendation**: I would recommend checking the current deposits before assigning the new limit.

```solidity
    function updateAssetDepositLimit(
        address asset,
        uint256 depositLimit
    )
        external
        onlyRole(LRTConstants.MANAGER)
        onlySupportedAsset(asset)
    {
+       uint256 currentDepositAmount = lrtDepositPool.getTotalAssetDeposits(asset);
+       if (depositLimit < currentDepositAmount) {
+           revert
+       }
        depositLimitByAsset[asset] = depositLimit;
        emit AssetDepositLimitUpdate(asset, depositLimit);
    }
```

---

### \[L-02\] `LRTDepositPool::addNodeDelegatorContractToQueue()` function should check if the inputted array includes the same addresses

[https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L162C1-L176C6](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L162C1-L176C6)

`addNodeDelegatorContractToQueue()` function takes an array parameter and pushes all the elements of the array into the `nodeDelegatorQueue` storage variable. While doing this, there is no check if these elements in the array are the same or not.

The biggest impact of this issue arises when calculating total assets deposits and getting the asset distribution data in the `getAssetDistributionData()` function. This function iterates through all the node delegators in the `nodeDelegatorQueue` and sums up all of their balances.  
[https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L71C1-L89C6](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L71C1-L89C6)

```solidity
    function getAssetDistributionData(address asset) {
        // skipped for brevity

        uint256 ndcsCount = nodeDelegatorQueue.length;
        for (uint256 i; i < ndcsCount;) {
            assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]); //@audit - it the array has the same delegator address in it, calculated balance will be greater than the actual deposit
            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
            unchecked {
                ++i;
            }
        }
    }
```

If the `nodeDelegatorQueue` array includes the same node delegator twice, all asset distribution data will be calculated incorrectly.

**Recommendation**: I would recommend adding sanity checks to the `addNodeDelegatorContractToQueue()` function and not allow the same addresses in the inputted array, even if it's a manager-only function.

---

### \[L-03\] `LRTDepositPool::updateMaxNodeDelegatorCount` should check the new count is not below the current delegator count

[https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L202C1-L205C6](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L202C1-L205C6)

```solidity
    function updateMaxNodeDelegatorCount(uint256 maxNodeDelegatorCount_) external onlyLRTAdmin {
        maxNodeDelegatorCount = maxNodeDelegatorCount_;
        emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount);
    }
```

Similar to L-01, this function also doesn't check the current status of the contract while updating a configuration parameter.

It is not mentioned in the docs that these updates will only performed to increase delegator counts. If the manager decides to decrease the `maxNodeDelegatorCount`, the function should check the current delegator count and should not allow breaking the invariant.

**Recommendation**: I would recommend checking the current status of the contract before assigning the new value.

```solidity
   function updateMaxNodeDelegatorCount(uint256 maxNodeDelegatorCount_) external onlyLRTAdmin {
+       // New value should be equal or bigger than the current node delegator count
+       require(maxNodeDelegatorCount_ >= nodeDelegatorQueue.length)
        maxNodeDelegatorCount = maxNodeDelegatorCount_;
        emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount);
    }
```

---

### \[L-04\] Deposited amounts in the EigenLayer strategy should be checked before updating the strategy for the asset

Users deposit in this protocol and the protocol deposits these funds to EigenLayer strategy contracts. There are 3 supported assets (rETH, cbETH, stETH) at the moment and every asset has one strategy contract.

The protocol also has a maximum deposit limit for each asset. An asset's total deposits are calculated by summing up the asset amounts in the pool, in node delegators and in the EigenLayer strategy.

The admin can update the strategy contract address for these assets.  
[https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L109C1-L122C6](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L109C1-L122C6)

```solidity
    function updateAssetStrategy(
        address asset,
        address strategy
    )
        external
        onlyRole(DEFAULT_ADMIN_ROLE)
        onlySupportedAsset(asset)
    {
        UtilLib.checkNonZeroAddress(strategy);
        if (assetStrategy[asset] == strategy) {
            revert ValueAlreadyInUse();
        }
        assetStrategy[asset] = strategy;
    }
```

As we can see above, this update is performed and the new strategy address is assigned without checking if there are already deposited assets in the previous strategy.

The biggest problem with that is the total asset deposits are calculated using the **current** EigenLayer strategy contract.  
[https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L84](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L84)  
[https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L121C1-L124C6](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L121C1-L124C6)

```solidity
file: NodeDelegator.sol
//@audit Strategy contracts can be changed for assets. If the strategy is changed for an asset (before withdrawing the balance from previous strategy), 
//       this function will return only the balance in the current strategy and the total deposits will be calculated lower than the actual.
    function getAssetBalance(address asset) external view override returns (uint256) { 
        address strategy = lrtConfig.assetStrategy(asset);
        return IStrategy(strategy).userUnderlyingView(address(this));
    }
```

If the strategy address is updated without withdrawing deposits from the previous strategy, the `totalAssetDeposits` will be calculated lower than the actual deposit amount. This will lead to the [`getAssetCurrentLimit()`](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L56) function returning a bigger value than the actual limit and breaking the maximum deposit invariant.

**Recommendation**: I would recommend checking if there are already deposited assets in the EigenLayer strategy, and withdrawing them before updating the strategy (*Or transferring deposits from the old strategy to the new strategy in EigenLayer, and updating the asset strategy in the same transaction with a multicall-like action*).

---

### \[L-05\] `ChainlinkPriceOracle::getAssetPrice()` function should check the stale price

[https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/oracles/ChainlinkPriceOracle.sol#L37C1-L39C6](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/oracles/ChainlinkPriceOracle.sol#L37C1-L39C6)

```solidity
    function getAssetPrice(address asset) external view onlySupportedAsset(asset) returns (uint256) {
        return AggregatorInterface(assetPriceFeed[asset]).latestAnswer();
    }
```

The current implementation accepts the Chainlink price without performing additional checks.

The protocol should check when this price is returned and also check the heartbeat of the oracle.

---

### \[NC-01\] `ChainlinkPriceOracle.sol` contract should use AggregatorV3Interface instead of AggregatorInterface

[https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/oracles/ChainlinkPriceOracle.sol#L11C1-L13C2](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/oracles/ChainlinkPriceOracle.sol#L11C1-L13C2)

```solidity
interface AggregatorInterface {
    function latestAnswer() external view returns (uint256);
}
```

The current implementation of the ChainlinkOracle.sol uses the old interface and should use the `AggregatorV3Interface`.

---

### \[NC-02\] It would be better to check the decimal precision of the returned values from oracles

According to the docs and contest channel on Discord, the Oracle prices are based on ETH prices and the ETH pairs will be used as price feed. It is also mentioned that the protocol is oracle agnostic, and other oracles than the Chainlink might be used.

The decimals of the ETH pairs in Chainlink is 18, and it works as expected in the current code.

However, all calculations will be incorrect if the protocol uses another oracle whose prices are in different decimal precision.

**Recommendation**: I would recommend checking the decimal value of the returned price, and normalizing the decimals if it is not 18.