https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L82
```
if (isSupportedAsset[asset]) {
            revert AssetAlreadySupported();
        }
```
```
        isSupportedAsset[asset] = true; 
```
No Need to check and write this , only checking this will save some gas
