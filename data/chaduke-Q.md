QA1. It might overwrite an existing key: 

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L156-L163

Mitigation:

```diff
 function _setToken(bytes32 key, address val) private {
        UtilLib.checkNonZeroAddress(val);
-        if (tokenMap[key] == val) {
+        if (tokenMap[key] != address(0)) {

            revert ValueAlreadyInUse();
        }
        tokenMap[key] = val;
        emit SetToken(key, val);
    }
```

QA2. LRTConfigRoleChecker.lrtConfigAddr() has the danger to lose the admin for the newly set lrtConfigAddr. 

[https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConfigRoleChecker.sol#L47-L50]

The function is only callable by the current LRTAdmin, which might not be the LRTAdmin for the newly set lrtConfigAddr. If an invalid lrtConfigAddr is set, then there is no way to correct it. 

Mitigation: ensure that the current LRTadmin will also be the LRTAdmin for the new lrtConfigAddr. After that, one can also transfer the LRTadmin to another use using a two-step procedure. 


QA3. There is no mechanism to check whether duplicate nodeDelegatorContract is added to the array or not. 

[https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L162-L176](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L162-L176)

Impact: getTotalAssetDeposits() might return the wrong total value since the balance for the duplicate nodeDelegatorContract will be doubly accounted. A user might not be able to deposit asset due to falsely inflated total deposit amount.

Mitigation: 
Introduce a mapping so that one can check whether we are adding a new nodeDelegatorContract or not to avoid adding duplicate. Otherwise, the limit ``maxNodeDelegatorCount`` might be wasted due to duplicates. 

QA4. LRTDepositPool.updateMaxNodeDelegatorCount() might decrease  ``maxNodeDelegatorCount``, but fails to check that the new ``maxNodeDelegatorCount_`` might result that the number of nodeDelegators already exceeds the new limit. 

[https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L202-L205](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L202-L205)

Mitigation: 
Check and ensure the new  ``maxNodeDelegatorCount_``  is not small than the number of current nodeDelegators. Otherwise, the function should revert. 

QA5. getAssetPrice() fails to check that assetPriceOracle[asset] == address(0). It is possible that the Oracle for the asset has not been set yet. 

[https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L45-L47](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L45-L47)

Mitigation: check and ensure assetPriceOracle[asset] != address(0).