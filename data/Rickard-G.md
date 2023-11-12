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