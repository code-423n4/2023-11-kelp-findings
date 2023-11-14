## Unnecessary use of multiple mapping for the same key

In `LRTDepositConfig.sol` there are three mapping with the same key of token address.

### Code Line

https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L15C4-L17C78

```js

// @audit use struct instead
    mapping(address token => bool isSupported) public isSupportedAsset;
    mapping(address token => uint256 amount) public depositLimitByAsset;
    mapping(address token => address strategy) public override assetStrategy;

```

 Instead of using multiple storage to store data corresponding to the same key, it can use a struct to store all data and update this struct when necessary. This way only one storage slot will be used saving a massive amount of gas, also it will be a lot easier to query the data this way.

### Example

```js

struct Asset {
   bool isSupportedAsset;
   uint256 depositLimit;
   address strategy; 
}

mapping(address token => Asset asset) public asset;

```