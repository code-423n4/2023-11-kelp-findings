## [NC-01] Unnecessarily checking for non-zero address
The addresses of tokens `stETH`, `rETH` and `cbETH` are unnecessarily checked for a non-zero address in the `initialize` function of the `LRTConfig` contract. This check is also performed in the `_setToken` function.

```solidity
UtilLib.checkNonZeroAddress(stETH);
UtilLib.checkNonZeroAddress(rETH);
UtilLib.checkNonZeroAddress(cbETH);
```

```solidity
 function _setContract(bytes32 key, address val) private {
        UtilLib.checkNonZeroAddress(val);

        .....
 }
```

## [NC-02] Typos in the codebase
1. Typo in the comment above `initialize` function of the `LRTConfig` contract.

```diff
    /// @param cbETH cbETH address -> coinbase staked eth
-   /// @param rsETH_ cbETH address 
+   /// @param rsETH_ rsETH_ address 
    function initialize(
        address admin,
        address stETH,
```

2. Typo in the naming of the event

```diff
-emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]); 
+emit NodeDelegatorAddedInQueue(nodeDelegatorContracts[i]); 
```

## [NC-03] Check return shares for successful deposit
The `depositIntoStrategy` function of the Eigen StrategyManager contract returns shares of a depositor. It can be checked if `shares != 0` for a successful deposit.

```diff
-IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);

+uint256 shares = IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);

+ if(shares == 0) revert UnsuccessfulDeposit();

```

## [NC-04] Check if new deposit limit is below total asset deposits
The Manager has the right to update the deposit limit for every asset. If the new value for the deposit limit is below the total asset deposits `getTotalAssetDeposits` (of the DepositPool, all NDCs and staked ethers in the EigenLayer protocol) a stack underflow will occur in the calculation of `getAssetCurrentLimit()`.

```solidity
return lrtConfig.depositLimitByAsset(asset) - getTotalAssetDeposits(asset);
```