### [L-01] Bytecode might not be compatible with all EVM-based chains
The pragma statement shows usage of 0.8.21 version of the Solidity compiler. This version (and every version after 0.8.19) will use the PUSH0 opcode, which is still not supported on some EVM-based chains, for example Arbitrum. Consider using version 0.8.19 so that the same deterministic bytecode can be deployed to all chains. 

### [L-02] Use of upgradeable tokens may result in improper accounting
According to Eigenlayer [documentation](https://github.com/Layr-Labs/eigenlayer-contracts/blob/75e59432d079c6f90d48d4e950a68c15867c82b2/src/contracts/strategies/StrategyBase.sol#L20), the use of fee-on-transfer tokens may result in accounting issues. For a token like cbETH, which has a proxy and implementation contract, if the implementation behind the proxy is changed, it can introduce features which might cause these issues, like adding a fee on transfer. Any accounting issues on Eigenlayer will translate to the kelp protocol.

### [L-03] Different price feeds decimals can cause accounting issues in `LRTDepositPool::getRsETHAmountToMint`
As of now, all of the supported assets have priceFeeds with 18 decimals. However, there is always a chance that in the future, there might be a new supported asset with a price feed with decimals =! 18. The calculation of the tokens to mint will be wrong as user will lose funds if decimals < 18. 
Context :
```javascript
    function getRsETHAmountToMint(
        address asset,
        uint256 amount
    )
        public
        view
        override
        returns (uint256 rsethAmountToMint)
    {
        // setup oracle contract
        address lrtOracleAddress = lrtConfig.getContract(LRTConstants.LRT_ORACLE);
        ILRTOracle lrtOracle = ILRTOracle(lrtOracleAddress);

        // calculate rseth amount to mint based on asset amount and asset exchange rate
@>      rsethAmountToMint = (amount * lrtOracle.getAssetPrice(asset)) / lrtOracle.getRSETHPrice();
    }
```
The (future) withdraw functionality will also be impacted.