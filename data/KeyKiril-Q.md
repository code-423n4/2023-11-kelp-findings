# Essential Early Checks

Key checks should be placed at the beginning part of a function logic where possible:

```solidity
    function getRSETHPrice() external view returns (uint256 rsETHPrice) {
    +    if (rsEthSupply == 0) {
    +       return 1 ether;
    +   }
        address rsETHTokenAddress = lrtConfig.rsETH();
        uint256 rsEthSupply = IRSETH(rsETHTokenAddress).totalSupply();


    -    if (rsEthSupply == 0) {
    -       return 1 ether;
    -   }


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