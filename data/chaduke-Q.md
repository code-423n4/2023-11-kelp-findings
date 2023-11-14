QA. It might overwrite an existing key: 

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
