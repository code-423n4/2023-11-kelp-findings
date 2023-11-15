## Summary

| severity       | Issues                                                                                                            | Instances |
| -------------- | ----------------------------------------------------------------------------------------------------------------- | --------- |
| [[L-0](#low0)] | Updating `maxNodeDelegatorCount` less than `nodeDelegatorQueue` should not be allowed.                            | 1         |
| [[L-1](#low1)] | `LRTConfig::updateAssetDepositLimit()` should not let asset deposit limit to set below the current deposit amount | 1         |

---

## Low

### [L-0] Updating `maxNodeDelegatorCount` less than `nodeDelegatorQueue` should not be allowed. <a id="low0"></a>

`LRTDepositPool::updateMaxNodeDelegatorCount(...)` does not check if the new max value is more than `nodeDelegatorQueue` length or not. If a value less than `nodeDelegatorQueue` length(or we can current number of active delegator) is used to call this function then it will not revert and will set that as `maxNodeDelegatorCount`. Now this will cause DoS in `addNodeDelegatorContractToQueue` due to below condition:

```Javascript
File: 2023-11-kelp/src/LRTDepositPool.sol

        if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
            revert MaximumNodeDelegatorCountReached();
        }
```

Github: [164-166](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L164C1-L166C10)

This condition will always met as the `maxNodeDelegatorCount` will always be less than `nodeDelegatorQueue.length + length`. In order for this to work again, the admin will have to call `LRTDepositPool::updateMaxNodeDelegatorCount(...)` again to update the max count.

#### PoC:

Test that proves that: [Test](https://github.com/Aamirusmani1552/kelp-audit/blob/a5566a18a540eaa34dd1c03e9c9a5c5df62a977e/test/LRTDepositPoolTest.t.sol#L530)

#### Recommendation:

It is recommended to add the following check:

```diff
File: 2023-11-kelp/src/LRTDepositPool.sol

    function updateMaxNodeDelegatorCount(uint256 maxNodeDelegatorCount_) external onlyLRTAdmin {
+        if(maxNodeDelegator > nodeDelegatorQueue.length){
+            revert InvalidMaxNodeDelegatorCount();
+        }

        maxNodeDelegatorCount = maxNodeDelegatorCount_;
        emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount);
    }
```

---

### [L-1] `LRTConfig::updateAssetDepositLimit(...)` should not let asset deposit limit to set below the current deposit amount <a id="low1"></a>

`LRTConfig::updateAssetDepositLimit(...)` does not check if the current asset Deposit limit is less than the current deposits for the asset. Because of this, the new limit will be set below current deposits and it will cause `LRTConfig::getAssetCurrentLimit(...)` to always revert due to underflow.

#### PoC:

Here is a test that proves that: [Test](https://github.com/Aamirusmani1552/kelp-audit/blob/a5566a18a540eaa34dd1c03e9c9a5c5df62a977e/test/LRTDepositPoolTest.t.sol#L228C5-L248C2)

#### Recommendation:

It is recommended to add the following check

```diff
File: File: 2023-11-kelp/src/LRTCofig.sol

    function updateAssetDepositLimit(
        address asset,
        uint256 depositLimit
    )
        external
        onlyRole(LRTConstants.MANAGER)
        onlySupportedAsset(asset)
    {
        // Assumed that `lrtDepositPool` is stored in the contract later
+        if(depositLimit < ILRTDepositPool(lrtDepositPool).getTotalAssetDeposits(asset)){
+            revert InvalidLimitForAssetDeposit();
+        }
        depositLimitByAsset[asset] = depositLimit;
        emit AssetDepositLimitUpdate(asset, depositLimit);
    }
```
