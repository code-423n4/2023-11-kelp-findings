# Suggest Event Emission

Suggest add updateAssetStrategy event in LRTConfig::updateAssetStrategy().
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