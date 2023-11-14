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