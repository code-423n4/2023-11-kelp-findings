# [G-01] Redundant variable in `LRTConfigRoleChecker.sol`

[File: LRTConfigRoleChecker.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConfigRoleChecker.sol#L27-L33)
```
    modifier onlyLRTAdmin() {
        bytes32 DEFAULT_ADMIN_ROLE = 0x00;
        if (!IAccessControl(address(lrtConfig)).hasRole(DEFAULT_ADMIN_ROLE, msg.sender)) {
            revert ILRTConfig.CallerNotLRTConfigAdmin();
        }
        _;
    }
```

Variable `DEFAULT_ADMIN_ROLE` is used only once, thus it does not need to be declared at all:

```
    modifier onlyLRTAdmin() {
        if (!IAccessControl(address(lrtConfig)).hasRole(0x00, msg.sender)) {
            revert ILRTConfig.CallerNotLRTConfigAdmin();
        }
        _;
    }
```


# [G-02] Redundant variables is `LRTOracle.sol`

[File: LRTOracle.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L52-L73)
```
 function getRSETHPrice() external view returns (uint256 rsETHPrice) {
        address rsETHTokenAddress = lrtConfig.rsETH();
        uint256 rsEthSupply = IRSETH(rsETHTokenAddress).totalSupply();

        if (rsEthSupply == 0) {
            return 1 ether;
        }

        uint256 totalETHInPool;
        address lrtDepositPoolAddr = lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL);

        address[] memory supportedAssets = lrtConfig.getSupportedAssetList();
        uint256 supportedAssetCount = supportedAssets.length;

        for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
            address asset = supportedAssets[asset_idx];
            uint256 assetER = getAssetPrice(asset);

            uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
            totalETHInPool += totalAssetAmt * assetER;

            unchecked {
                ++asset_idx;
            }
        }

        return totalETHInPool / rsEthSupply;
    }
```


Variables `rsETHTokenAddress`, `assetER`, `totalAssetAmt` are used only once, thus they don't need to be declared at all:

```
 function getRSETHPrice() external view returns (uint256 rsETHPrice) {
        
        uint256 rsEthSupply = IRSETH(lrtConfig.rsETH()).totalSupply();

        if (rsEthSupply == 0) {
            return 1 ether;
        }

        uint256 totalETHInPool;
        address lrtDepositPoolAddr = lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL);

        address[] memory supportedAssets = lrtConfig.getSupportedAssetList();
        uint256 supportedAssetCount = supportedAssets.length;

        for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
            address asset = supportedAssets[asset_idx];
            totalETHInPool += ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset) * getAssetPrice(asset);

            unchecked {
                ++asset_idx;
            }
        }

        return totalETHInPool / rsEthSupply;
    }
```

# [G-03] Do not perform `depositIntoStrategy()` when deposited amount is 0



[File: NodeDelegator.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L63-L67)
```
 uint256 balance = token.balanceOf(address(this));

        emit AssetDepositIntoStrategy(asset, strategy, balance);

        IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
```

`IEigenStrategyManager(eigenlayerStrategyManagerAddress.depositIntoStrategy()` deposits `balance` of `token` into the specified `strategy`.

However, whenever `balance` is 0, performing that function basically does nothing. 

Checking non-zero transfer values can avoid an expensive external call and save gas.

Moreover, we can change the order of operations in function, to perform balance-check quicker.

Function can be rewritten from:

```
        address strategy = lrtConfig.assetStrategy(asset);
        IERC20 token = IERC20(asset);
        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

        uint256 balance = token.balanceOf(address(this));

        emit AssetDepositIntoStrategy(asset, strategy, balance);

        IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
```

to:

```
        IERC20 token = IERC20(asset);
        uint256 balance = token.balanceOf(address(this));

        if (balance == 0) revert DepositingZeroAmount();

        address strategy = lrtConfig.assetStrategy(asset);
        
        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

        emit AssetDepositIntoStrategy(asset, strategy, balance);

        IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
```

That way, when `balance == 0`, we don't spend gas not only on useless `depositIntoStrategy()`, but we also don't spend gas on `assetStrategy()` and `lrtConfig.getContract()` calls.


# [G-04] Redundant variables in `NodeDelegator.sol`


* Function `transferBackToLRTDepositPool()`:

Variable `lrtDepositPool` is used only once, which means it doesn't need to be declared at all:

[File: NodeDelegatotor.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L84-L88)
```
        address lrtDepositPool = lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL);

        if (!IERC20(asset).transfer(lrtDepositPool, amount)) {
            revert TokenTransferFailed();
        }
```

can be changed to:

```

        if (!IERC20(asset).transfer(lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL), amount)) {
            revert TokenTransferFailed();
        }
```

* Function `getAssetBalances()`:

Variable `eigenlayerStrategyManagerAddress` is used only once, which means it doesn't need to be declared at all.


[File: NodeDelegatotor.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L100-L103)
```
        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

        (IStrategy[] memory strategies,) =
            IEigenStrategyManager(eigenlayerStrategyManagerAddress).getDeposits(address(this));
```

can be changed to:

```
 (IStrategy[] memory strategies,) =
            IEigenStrategyManager(lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER)).getDeposits(address(this));
```

* Function `getAssetBalance()`:

Variable `strategy` is used only once, which means it doesn't need to be declared at all:


[File: NodeDelegatotor.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L122-L123)
```
        address strategy = lrtConfig.assetStrategy(asset);
        return IStrategy(strategy).userUnderlyingView(address(this));
```

can be changed to:

```
        return IStrategy(lrtConfig.assetStrategy(asset)).userUnderlyingView(address(this));
```


# [G-05] In `transferBackToLRTDepositPool()` amount should be checked for `0` before calling `transfer()`

Checking non-zero transfer values can avoid an expensive external call and save gas.

[File: NodeDelegatotor.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L84-L88)
```
        address lrtDepositPool = lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL);

        if (!IERC20(asset).transfer(lrtDepositPool, amount)) {
            revert TokenTransferFailed();
        }
```


# [G-06] Use `do-while` loops instead of `for-loops`

Solidity do-while loops are more gas efficient than for loops. 


[File: NodeDelegator.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L109-L115)
```
        for (uint256 i = 0; i < strategiesLength;) {
            assets[i] = address(IStrategy(strategies[i]).underlyingToken());
            assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
            unchecked {
                ++i;
            }
        }
```

[File: LRTDepositPool.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L82-L87)
```
        for (uint256 i; i < ndcsCount;) {
            assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
            unchecked {
                ++i;
            }
        }
```

Moreover, since above loops do not perform any advanced operations (assigning data to an array and calculating the sum of array)) - we can easily re-write them, so they will count down.


Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable - [reference](https://code4rena.com/reports/2023-08-pooltogether#g02-counting-down-in-for-statements-is-more-gas-efficient)



# [G-07] Redundant variables in `LRTDepositPool.sol`


* Function `getRsETHAmountToMint()`:

Variable `lrtOracleAddress` is used only once, which means it doesn't need to be declared at all.

[File: LRTDepositPool.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L105-L106)
```
        address lrtOracleAddress = lrtConfig.getContract(LRTConstants.LRT_ORACLE);
        ILRTOracle lrtOracle = ILRTOracle(lrtOracleAddress);
```

can be changed to:

```
        ILRTOracle lrtOracle = ILRTOracle(lrtConfig.getContract(LRTConstants.LRT_ORACLE));
```


* Function `_mintRsETH()`:

Variable `rsethToken` is used only once, which means it doesn't need to be declared at all.

[File: LRTDepositPool.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L154-L156)
```
        address rsethToken = lrtConfig.rsETH();
        // mint rseth for user
        IRSETH(rsethToken).mint(msg.sender, rsethAmountToMint);
```

can be changed to:

```
        // mint rseth for user
        IRSETH(rsethToken).mint(msg.sender, lrtConfig.rsETH());
```

* Function `transferAssetToNodeDelegator()`:

Variable `nodeDelegator` is used only once, which means it doesn't need to be declared at all.

[File: LRTDepositPool.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L193-L194)
```
        address nodeDelegator = nodeDelegatorQueue[ndcIndex];
        if (!IERC20(asset).transfer(nodeDelegator, amount)) {
```

can be changed to:

```
if (!IERC20(asset).transfer(nodeDelegatorQueue[ndcIndex], amount)) {
```


# [G-08] `initialize()` in `LRTConfig.sol` does not need to call `UtilLib.checkNonZeroAddress` on `stEth`, `rETH`, `cbETH`:

[File: LRTConfig.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L52-L55)
```
        UtilLib.checkNonZeroAddress(stETH);
        UtilLib.checkNonZeroAddress(rETH);
        UtilLib.checkNonZeroAddress(cbETH);
```

Above calls are redundant. `initialize()` calls `_setToken()` on `stEth`, `rETH`, `cbETH`:

[File: LRTConfig.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L58-L60)
```
        _setToken(LRTConstants.ST_ETH_TOKEN, stETH);
        _setToken(LRTConstants.R_ETH_TOKEN, rETH);
        _setToken(LRTConstants.CB_ETH_TOKEN, cbETH);
```

As demonstrated above, `initialize()` firstly checks for `address(0)` by performing `UtilLib.checkNonZeroAddress()` on `stEth`, `rETH`, `cbETH`, and then, calls `_setToken()` on that addresses.

However, function `_setToken()` already implements a check for `address(0)`

[File: LRTConfig.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L156-L157)
```
    function _setToken(bytes32 key, address val) private {
        UtilLib.checkNonZeroAddress(val);
```

thus doing it one more time in `initialize()` is redundant.

```
        UtilLib.checkNonZeroAddress(stETH);
        UtilLib.checkNonZeroAddress(rETH);
        UtilLib.checkNonZeroAddress(cbETH);
```

can be removed from `initialize()`.


# [G-09] Use constants instead of `type(uintX).max`

When you use type(uintX).max - it may result in higher gas costs because it involves a runtime operation to calculate the `type(uintX).max` at runtime. This calculation is performed every time the expression is evaluated.

To save gas, it is recommended to use constants to represent the maximum value. Declaration of a constant with the maximum value means that the value is known at compile-time and does not require any runtime calculations.


[File: NodeDelegator.sol](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L45)
```
 IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);
```