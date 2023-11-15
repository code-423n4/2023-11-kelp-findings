## Systemic risks
`RSETH` represents the total price of different assets, and if one asset experiences significant fluctuations, other assets will also be affected.

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L91-L110
```
    /// @notice View amount of rsETH to mint for given asset amount
    /// @param asset Asset address
    /// @param amount Asset amount
    /// @return rsethAmountToMint Amount of rseth to mint
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
        rsethAmountToMint = (amount * lrtOracle.getAssetPrice(asset)) / lrtOracle.getRSETHPrice();
    }
```
## Other recommendations
Due to design, there is currently no withdraw function, which results in a significant reduction in attack surface area.
After improving the relevant functions, it is strongly recommended to hold another contest.

## learnings from the audit

`LST` has great potential,and `Eignelayer` is very interesting,I will continue to study after the contest.
```
https://github.com/Layr-Labs/eigenlayer-contracts
```


### Time spent:
28 hours