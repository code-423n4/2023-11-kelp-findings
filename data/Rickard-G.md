## [G-01] Remove the `initializer` modifier
If we can just ensure that the `initialize()` function can only be called from within the constructor, we shouldn’t need to worry about it getting called again.
````solidity
src/LRTConfig.sol
41:    function initialize(
42:        address admin,
43:        address stETH,
44:        address rETH,
45:        address cbETH,
46:        address rsETH_
47:    )
48:        external
49:        initializer
50:    {
````
[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L41-L50](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L41-L50)
````solidity
src/LRTDepositPool.sol
31:    function initialize(address lrtConfigAddr) external initializer {
````
[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTDepositPool.sol#L31](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTDepositPool.sol#L31)
````solidity
src/LRTOracle.sol
29:    function initialize(address lrtConfigAddr) external initializer {
````
[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTOracle.sol#L29](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTOracle.sol#L29)
````solidity
src/NodeDelegator.sol
26:    function initialize(address lrtConfigAddr) external initializer {
````
[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/NodeDelegator.sol#L26](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/NodeDelegator.sol#L26)
````solidity
src/RSETH.sol
32:    function initialize(address admin, address lrtConfigAddr) external initializer {
````
[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/RSETH.sol#L32](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/RSETH.sol#L32)
````solidity
src/oracles/ChainlinkPriceOracle.sol
27:    function initialize(address lrtConfig_) external initializer {
````
[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/oracles/ChainlinkPriceOracle.sol#L27](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/oracles/ChainlinkPriceOracle.sol#L27)
## [G-02] Multiple accesses of a mapping/array should use a local variable cache
Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it.

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `someMap[someIndex]`, save its reference like this: `SomeStruct storage someStruct = someMap[someIndex]` and use it.
## Proof Of Concept
````solidity
src/LRTConfig.sol
29:        if (!isSupportedAsset[asset]) {
82:        if (isSupportedAsset[asset]) {
85:        isSupportedAsset[asset] = true;
````
[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L29](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L29)

[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L82](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L82)

[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L85](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L85)
````solidity
src/LRTConfig.sol
87:        depositLimitByAsset[asset] = depositLimit;
102:       depositLimitByAsset[asset] = depositLimit;
````
[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L87](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L87)

[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L102](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L102)
````solidity
src/LRTConfig.sol
118:        if (assetStrategy[asset] == strategy) {
121:        assetStrategy[asset] = strategy;
````
[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L118](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L118)

[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L121](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L121)
````solidity
src/LRTConfig.sol
158:        if (tokenMap[key] == val) {
161:        tokenMap[key] = val;
````
[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L158](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L158)

[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L161](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L161)
````solidity
src/LRTConfig.sol
174:        if (contractMap[key] == val) {
177:        contractMap[key] = val;
````
[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L174](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L174)

[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L177](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L177)
````solidity
src/LRTOracle.sol
46:        return IPriceFetcher(assetPriceOracle[asset]).getAssetPrice(asset);
97:        assetPriceOracle[asset] = priceOracle;
````
[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTOracle.sol#L46](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTOracle.sol#L46)

[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTOracle.sol#L97](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTOracle.sol#L97)
````solidity
src/oracles/ChainlinkPriceOracle.sol
38:        return AggregatorInterface(assetPriceFeed[asset]).latestAnswer();
47:        assetPriceFeed[asset] = priceFeed;
````
[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/oracles/ChainlinkPriceOracle.sol#L38](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/oracles/ChainlinkPriceOracle.sol#L38)

[https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/oracles/ChainlinkPriceOracle.sol#L47](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/oracles/ChainlinkPriceOracle.sol#L47)
## [G-03] Multiple `address/bytes32` mappings can be combined into a single mapping of an `address/bytes32` to a `struct`, where appropriate
## Summary
````solidity
src/LRTConfig.sol
13:    mapping(bytes32 tokenKey => address tokenAddress) public tokenMap;
14:    mapping(bytes32 contractKey => address contractAddress) public contractMap;
````
[https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L13-L14](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L13-L14)
````solidity
src/LRTConfig.sol
15:    mapping(address token => bool isSupported) public isSupportedAsset;
16:    mapping(address token => uint256 amount) public depositLimitByAsset;
17:    mapping(address token => address strategy) public override assetStrategy;
````
[https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L15-L17](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L15-L17)