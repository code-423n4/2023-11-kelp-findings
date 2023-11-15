## [01]Avoid using deprecated methods

ChainlinkPriceOracle used the latestAnswer() function when requesting a price. But this method has already been enabled, consider using latestRoundData instead.

File:ChainlinkPriceOracle.sol
GitHub:[[37](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/oracles/ChainlinkPriceOracle.sol#L37)]
```solidity
    function getAssetPrice(address asset) external view onlySupportedAsset(asset) returns (uint256) {
        return AggregatorInterface(assetPriceFeed[asset]).latestAnswer();
    }
```

## [02]Return value type mismatch

In the chainlink contract AggregatorProxy, calling latestAnswer() returns the int256 type, while the project requests lastAnswer() and returns the uint256 type.  
Although this will not cause security issues, consider maintaining consistent return value types.
File:ChainlinkPriceOracle.sol
GitHub:[[37](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/oracles/ChainlinkPriceOracle.sol#L37)]
```solidity
    function getAssetPrice(address asset) external view onlySupportedAsset(asset) returns (uint256) {
        return AggregatorInterface(assetPriceFeed[asset]).latestAnswer();
    }
```

File:chainlink/contracts/src/v0.7/dev/AggregatorProxy.sol
GitHub:[[41](https://github.com/smartcontractkit/chainlink/blob/66d9e23219831c8ecd937431ead528dc0e9afcbb/contracts/src/v0.7/dev/AggregatorProxy.sol#L41C15-L41C15)]
```solidity
    function latestAnswer() public view virtual override returns (int256 answer) {
        return s_currentPhase.aggregator.latestAnswer();
    }
```


