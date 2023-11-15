Low - 1

Incorrect check for `_setToken` and `_setContract` which overwrite the existing key's value

`LRTConfig` contract has the following logic to check and update the new token and contract.

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L156-L163

```solidity
    function _setToken(bytes32 key, address val) private {
        UtilLib.checkNonZeroAddress(val);
        if (tokenMap[key] == val) { --------------->>>audit - not sufficient. lead to overwriting existing key
            revert ValueAlreadyInUse();
        }
        tokenMap[key] = val;
        emit SetToken(key, val);
    }
```

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L172-L179

```solidity

    function _setContract(bytes32 key, address val) private {
        UtilLib.checkNonZeroAddress(val);
        if (contractMap[key] == val) { --------------------->> audit - not sufficient. lead to overwriting existing key
            revert ValueAlreadyInUse();
        }
        contractMap[key] = val;
        emit SetContract(key, val);
    }
```

Recommendation:

Update the code as shown below

```solidity
    function _setToken(bytes32 key, address val) private {
        UtilLib.checkNonZeroAddress(val);

        require(tokenMap[key] == address(0)); ----------->>> fix        

        if (tokenMap[key] == val) { ------------->>>remove this check
            revert ValueAlreadyInUse();
        }
        tokenMap[key] = val;
        emit SetToken(key, val);
    }
```




