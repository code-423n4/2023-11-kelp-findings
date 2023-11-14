## [LOW -01] - getRsETHAmountToMint() should check that asset address suplied is a supported asset 

### Description
in `getRsETHAmountToMint()` there are no checks to see that the address supplied as `asset` is a supported asset. A unsupported asset can be supplied to the function logic, This can be misleading to users who use the function to predict the amounts of RsETH they can mint. Thus this is an incomplete validation of inputs to the view function. 

```
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
         // 2e18 * 998918684011422300 / 1.998e+18 = 0.9999186e+18 ; 
        rsethAmountToMint = (amount * lrtOracle.getAssetPrice(asset)) / lrtOracle.getRSETHPrice();
    }
```

### Recommended Mitigation
add the modifier  `onlySupportedAsset(asset)` to the function code.
