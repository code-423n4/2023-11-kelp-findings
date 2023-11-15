1. Use uint256(1)/uint256(2) instead of true/false to save gas for changes
Avoids a Gsset (20000 gas) when changing from false to true, after having been true in the past. 
```
mapping(address token => bool isSupported) public isSupportedAsset;
```
LRTConfig.sol [L15](https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/LRTConfig.sol#L15)

2. Initializers can be marked payable
```
    function initialize(
```
LRTConfig.sol [L41](https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/LRTConfig.sol#L41)
```
 function initialize(address lrtConfigAddr) external initializer {
```
LRTDepositPool.sol [L31](https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/LRTDepositPool.sol#L31)

```
    function initialize(address lrtConfigAddr) external initializer {
```
LRTOracle.sol [L29](https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/LRTOracle.sol#L29)

```
    function initialize(address lrtConfigAddr) external initializer {
```
NodeDelegator.sol [L26](https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/NodeDelegator.sol#L26)

```
    function initialize(address admin, address lrtConfigAddr) external initializer {
```
RSETH.sol [L32](https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/RSETH.sol#L32)

```
    function initialize(address lrtConfig_) external initializer {
```
ChainlinkPriceOracle.sol [L27](https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/oracles/ChainlinkPriceOracle.sol#L27)