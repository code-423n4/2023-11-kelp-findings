# 1. Not considering Strategy deposit limit
## Summary
The eigenlayer strategy contracts have deposit limitations:
```solidity
    /// The maximum deposit (in underlyingToken) that this strategy will accept per deposit
    uint256 public maxPerDeposit;


    /// The maximum deposits (in underlyingToken) that this strategy will hold
    uint256 public maxTotalDeposits;
```
Which currently are [100000000000000000000000](https://etherscan.io/address/0x54945180dB7943c0ed0FEE7EdaB2Bd24620256bc#readProxyContract#F3), the problem here is NodeDelegator doesn't consider this and send the whole balance to strategies which may cause revert.
```solidity
        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

        uint256 balance = token.balanceOf(address(this));//@audit add amount instead of balance

        emit AssetDepositIntoStrategy(asset, strategy, balance);

        IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
```
## Recommendations
use amount as an argument instead of the balance of the contract
# 2. Not considering token decimals
## Summary
in share calculation protocol assume that all assets have 18 decimals but in couldn't be true in the future, asset with different decimals could result in wrong share calculation.
```solidity
        rsethAmountToMint = (amount * lrtOracle.getAssetPrice(asset)) / lrtOracle.getRSETHPrice();
```
```solidity
        return IPriceFetcher(assetPriceOracle[asset]).getAssetPrice(asset);
```
## Recommendations
add checks, to check token decimals.