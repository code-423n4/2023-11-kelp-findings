## setting all of the tokens with _setToken(), but doesn't set the other constants 

https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L58-L61

The tokens are set, but the other constants are not and the owner needs to set them manually, it should be set in the initializer.

## in updateAssetDepositLimit there is no check if the new limiter is already reached

https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L94-L104

in the future, it would be a good practice to check that, because it can lead to some issues