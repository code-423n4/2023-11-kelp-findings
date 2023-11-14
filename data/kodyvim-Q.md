## Missing Invariant check on pause functions
Pause should be callable when protocol is not in paused state already and vice versa for unpaused.
One Instance:
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L208
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L213
```solidity
/// @dev Triggers stopped state. Contract must not be paused.
    function pause() external onlyLRTManager {
        _pause();
    }

    /// @dev Returns to normal state. Contract must be paused
    function unpause() external onlyLRTAdmin {
        _unpause();
    }
```
other instances:
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L102
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L107
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L127
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L132
## Recommendation
Add [whenNotPaused](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/3d4c0d5741b131c231e558d7a6213392ab3672a5/contracts/security/PausableUpgradeable.sol#L49) to `pause()` and [whenPaused](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/3d4c0d5741b131c231e558d7a6213392ab3672a5/contracts/security/PausableUpgradeable.sol#L61) modifier to `unpause()`.

## No mechanism to nonsupport assets.
With the current implementation, once an asset is supported it cannot be unsupported without an upgrade.
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L80
```solidity
function _addNewSupportedAsset(address asset, uint256 depositLimit) private {
        UtilLib.checkNonZeroAddress(asset);
        if (isSupportedAsset[asset]) {
            revert AssetAlreadySupported();
        }
        isSupportedAsset[asset] = true;
        supportedAssetList.push(asset);
        depositLimitByAsset[asset] = depositLimit;
        emit AddedNewSupportedAsset(asset, depositLimit);
    }
```
## Recommendation
Add a mechanism to unsupport assets with proper access control.
