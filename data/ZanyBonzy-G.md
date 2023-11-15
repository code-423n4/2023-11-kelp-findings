### 1. Use uint256(1)/uint256(2) instead of true/false to save gas for changes
Avoids a Gsset (20000 gas) when changing from false to true, after having been true in the past. 
```
mapping(address token => bool isSupported) public isSupportedAsset;
```
LRTConfig.sol [L15](https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/LRTConfig.sol#L15)

### 2. Initializers can be marked payable

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


### 3. Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address, especially if you need to use the same address multiple times in your contract.

The reason for this, is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

```
        assetLyingInDepositPool = IERC20(asset).balanceOf(address(this));
```
LRTDepositPool.sol [L79](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L79)

```
        if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) {
```
LRTDeposit.sol [L136](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L136)

```
        uint256 balance = token.balanceOf(address(this));
```
NodeDelegator.sol [L63](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L63)
```
           IEigenStrategyManager(eigenlayerStrategyManagerAddress).getDeposits(address(this));
```
NodeDelegator.sol [L103](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L103)
```
            assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
```
NodeDelegator.sol [L111](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L111)
```
        return IStrategy(strategy).userUnderlyingView(address(this));
```
NodeDelegator.sol [L123](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L123)


### 4. `x = x + y` is cheaper than `x += y;`

```
            totalETHInPool += totalAssetAmt * assetER;
```
LRTOracle.sol [L71](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L71)

```
            assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).
```
LRTDepositPool.sol [83-84](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L82C42-L84C101)