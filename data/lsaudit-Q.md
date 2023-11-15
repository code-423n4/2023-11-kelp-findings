# [QA-01]  `updatePriceFeedFor()` emits an event even when the priceFeed has not been changed

[File: ChainlinkPriceOracle.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/oracles/ChainlinkPriceOracle.sol#L45-L49)
```
    function updatePriceFeedFor(address asset, address priceFeed) external onlyLRTManager onlySupportedAsset(asset) {
        UtilLib.checkNonZeroAddress(priceFeed);
        assetPriceFeed[asset] = priceFeed;
        emit AssetPriceFeedUpdate(asset, priceFeed);
    }
```

Function `updatePriceFeedFor()` does not verify if provided `priceFeed` is not already set in `assetPriceFeed[asset]`.
If `onlyLTRManager` provides `priceFeed` which was set before (`assetPriceFeed[asset] == priceFeed`), function will still emit an event.
Emitting this event may be very misleading to the end user - thus he/she may think that the price was updated, even though, it wasn't.



The same issue occurs in `LRTOracle.sol`:

[File: LRTOracle.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L88-L99)
```
 function updatePriceOracleFor(
        address asset,
        address priceOracle
    )
        external
        onlyLRTManager
        onlySupportedAsset(asset)
    {
        UtilLib.checkNonZeroAddress(priceOracle);
        assetPriceOracle[asset] = priceOracle;
        emit AssetPriceOracleUpdate(asset, priceOracle);
    }
```

It's a good practice to perform additional check which verifies if the value is really being updated:

```
if (assetPriceFeed[asset] == priceFeed) revert ValueAlreadyInUse();
```

You can verify how it's done in `LRConfig.sol`. For example here:

[File: LRTConfig.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L118)
```
 if (assetStrategy[asset] == strategy) {
            revert ValueAlreadyInUse();
    }
```



# [QA-02] Override `renounceRole()` in `RSETH.sol`

`RSETH` inherits from `AccessControlUpgradeable`, which implies, that it implements `renounceRole()` function. `renounceRole()` allows accounts to renounce granted roles belonging to them.

The `DEFAULT_ADMIN_ROLE` role is granted to `admin` address when function `initialize()` is called. This role is needed for proper contract functioning. E.g. role `DEFAULT_ADMIN_ROLE` allows to grant others with `BURNER_ROLE` and `MINTER_ROLE` - roles needed to mint and burn rsETH.
Moreover - `DEFAULT_ADMIN_ROLE` is the only role which allows to unpause a contract (which can be paused by `onlyLRTManager`).

This implies, that whenever `admin` mistakenly calls `renounceRole()`, protocol will be left with no `DEFAULT_ADMIN_ROLE` and may not be functioning properly.

It's a good security practice to override `renounceRole()`, so it won't be called by mistake.



# [QA-03] Emit `AssetDepositIntoStrategy()` event after performing `depositIntoStrategy()`

[File: NodeDelegator.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L65-L67)
```
        emit AssetDepositIntoStrategy(asset, strategy, balance);

        IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
```

The whole code-base follows the coding-pattern: action -> event.
In every implemented function, we have noticed, that the action is being performed first, then event is being emitted. The only exception for that rule was noticed in `depositAssetIntoStrategy()` - when event `AssetDepositIntoStrategy()` is emitted before calling `depositIntoStrategy()`.

Sometimes moving event up allows to save gas (we don't need to declare additional variables used in events) - however, this is not the case in the following example.

Our recommendation is to follow previously defined pattern and emit `AssetDepositIntoStrategy()` after performing `depositIntoStrategy()`.


# [QA-04] Improve code readability by defining a constant for default max node delagator count in `LRTDepositPool.sol`

[File: LRTDepositPool.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L35)
```
maxNodeDelegatorCount = 10;
```

Function `initialize()` sets `maxNodeDelegatorCount` to 10, which should be considered as default number of max node delegator count (which might be updated later by calling `updateMaxNodeDelegatorCount()`).

However, instead of using numbers, the code readability would be much clearer, when this value would be defined as constant:

```
uint256 private constant DEFAULT_MAX_NODE_DELEGATOR_COUNT = 10;

function initialize(address lrtConfigAddr) external initializer {
        UtilLib.checkNonZeroAddress(lrtConfigAddr);
        __Pausable_init();
        __ReentrancyGuard_init();
        maxNodeDelegatorCount = DEFAULT_MAX_NODE_DELEGATOR_COUNT;
        lrtConfig = ILRTConfig(lrtConfigAddr);
        emit UpdatedLRTConfig(lrtConfigAddr);
    }
```


# [QA-05] Use more descriptive variable naming in `LRTDepositPool.sol`

[File: LRTDepositPool.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L163-L168)
```
        uint256 length = nodeDelegatorContracts.length;
        if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
            revert MaximumNodeDelegatorCountReached();
        }

        for (uint256 i; i < length;) {
```

When calling `addNodeDelegatorContractToQueue()`, we provide `nodeDelegatorContracts` - an array of NodeDelegator contract addresses.
Function verifies if current number of NodeDeleagor contract addresses (`nodeDelegatorContracts`) and provided NodeDelegator contract addresses in parameter `nodeDelegatorContracts`, does not exceed `maxNodeDelegatorCount`.

It assigns `nodeDelegatorContracts.length` to variable `length`. However, since the code uses two different arrays: (`nodeDelegatorContracts` and `nodeDelegatorContracts`), this variable naming may be confusing in later part of the codebase. It will be much better to change the name of `length` to straightforwardly state that it's the length of `nodeDelegatorContracts`:

```
uint256 nodeDelegatorContractsLength = nodeDelegatorContracts.length;
```


# [QA-05] `updateMaxNodeDelegatorCount()` in `LRTDepositPool.sol` emits an event, even when `maxNodeDelegatorCount` has not been changed.

[File: LRTDepositPool.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L202-L205)
```
    function updateMaxNodeDelegatorCount(uint256 maxNodeDelegatorCount_) external onlyLRTAdmin {
        maxNodeDelegatorCount = maxNodeDelegatorCount_;
        emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount);
    }
```

Function `updateMaxNodeDelegatorCount()` does not verify if provided `maxNodeDelegatorCount_` is different than the one previously set. 
If `onlyLRTAdmin` provides `maxNodeDelegatorCount_` with the value which was set before (`maxNodeDelegatorCount_ == maxNodeDelegatorCount`), function will still emit an event.
Emitting this event may be very misleading to the end user - thus he/she may think that the `maxNodeDelegatorCount` was updated, even though, it wasn't.


It's a good practice to perform additional check which verifies if the value is really being updated:

```
if (maxNodeDelegatorCount == maxNodeDelegatorCount_) revert ValueAlreadyInUse();
```

You can verify how it's done in `LRConfig.sol`. For example here:

[File: LRTConfig.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L118)
```
 if (assetStrategy[asset] == strategy) {
            revert ValueAlreadyInUse();
    }
```



# [N-01] Typos/comments-coding-style consistency

* `LRTConstants.sol`

```
18:     //Roles
```

Every comment describing set of variables in `LRTConstants.sol` is lowercased, e.g.: `//contracts`, `//tokens`.
Keep your comment-coding-style consistent by changing `//Roles` to `//roles`

* `LRTConstants.sol`
```
5:    //tokens
6:    //rETH token
8:    //stETH token
10:   //cbETH token
13:   //contracts
18:   //Roles
```

Write comment after white-space character: `// Comment`, instead of `//Comment`.

* `LRTConfigRoleChecker.sol`
```
46:     /// @param lrtConfigAddr the new LRT config contract Address
```
Typo: `Address` should be lowercased - `address`

* `ChainlinkPriceOracle.sol`
```
16: /// @notice contract that fetches the exchange rate of assets from chainlink price feeds
```
Typo: `chainlink` should be changed to `Chainlink`

* `RSETH.sol`

```
59:    /// @dev Only callable by LRT config manager. Contract must NOT be paused.
```
None of the previous comment contains the dot at the end of line.
Keep your comment-coding-style consistent, by removing dot at the end.

* `LRTOracle.sol.sol`
```
50:     /// @dev calculates based on stakedAsset value received from eigen layer

```
Typo: `eigen layer` should be changed to `EigenLayer`.