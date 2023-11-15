## Gas Optimization Report for Kelp DAO

### Use Smaller Integer Types for State Variables

Smaller integer types can reduce gas costs significantly when used in state variables.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L16

**Instances:**
- `depositLimitByAsset` mapping uses `uint256` for deposit limits.

```solidity
mapping(address token => uint256 amount) public depositLimitByAsset;
```

**Proposed Changes:**
If the expected range of `depositLimitByAsset` values is within the limits of a smaller integer type, consider using `uint128` or another smaller type.

```solidity
mapping(address token => uint128 amount) public depositLimitByAsset;
```

### Caching Results of External Functions

Repeatedly calling the same external function in a transaction can be optimized by caching its result in a local variable.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L80

**Instances:**
- Multiple checks of `isSupportedAsset[asset]` in `_addNewSupportedAsset` function.

```solidity
function _addNewSupportedAsset(address asset, uint256 depositLimit) private {
    UtilLib.checkNonZeroAddress(asset);
    if (isSupportedAsset[asset]) {
        revert AssetAlreadySupported();
    }
    // ...
}
```

**Proposed Changes:**
Cache the value of `isSupportedAsset[asset]` in a local variable to reduce gas costs.

```solidity
function _addNewSupportedAsset(address asset, uint256 depositLimit) private {
    UtilLib.checkNonZeroAddress(asset);
    bool assetSupported = isSupportedAsset[asset];
    if (assetSupported) {
        revert AssetAlreadySupported();
    }
    // ...
}
```

### Using Immutable for Fixed Values

Immutable variables are cheaper to access than regular state variables.

**Instances:**
- `rsETH` address is a regular state variable.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L21

```solidity
address public rsETH;
```

**Proposed Changes:**
If `rsETH` is set once and does not change, consider making it immutable.

```solidity
address public immutable rsETH;
```

### Inlining Simple Checks

Inlining simple functions can save gas, especially if they are called frequently.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L81

**Instances:**
- Frequent calls to `UtilLib.checkNonZeroAddress`.

```solidity
UtilLib.checkNonZeroAddress(asset);
```

**Proposed Changes:**
Inline the `checkNonZeroAddress` function if it is simple, for instance:

```solidity
require(asset != address(0), "Zero address not allowed");
```

### Use Smaller Integer Types for State Variables

Reducing the size of integer types can help save gas, particularly for frequently accessed or modified state variables.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L20

**Instances:**
- `maxNodeDelegatorCount` is a `uint256`.

```solidity
uint256 public maxNodeDelegatorCount;
```

**Proposed Changes:**
If the value of `maxNodeDelegatorCount` never exceeds the range of a smaller integer type, consider using `uint128` or another smaller type.

```solidity
uint128 public maxNodeDelegatorCount;
```

### Caching Results of External Functions

Caching the results of external function calls can be more gas-efficient than making repeated calls.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L95

**Instances:**
- The `getRsETHAmountToMint` function makes multiple oracle calls.

```solidity
function getRsETHAmountToMint(address asset, uint256 amount) public view override returns (uint256 rsethAmountToMint) {
    address lrtOracleAddress = lrtConfig.getContract(LRTConstants.LRT_ORACLE);
    ILRTOracle lrtOracle = ILRTOracle(lrtOracleAddress);
    // ...
}
```

**Proposed Changes:**
Cache the oracle address and the returned values from the oracle if they are used multiple times within the same function or transaction.

```solidity
function getRsETHAmountToMint(address asset, uint256 amount) public view override returns (uint256 rsethAmountToMint) {
    ILRTOracle lrtOracle = ILRTOracle(lrtConfig.getContract(LRTConstants.LRT_ORACLE));
    uint256 assetPrice = lrtOracle.getAssetPrice(asset);
    uint256 rsethPrice = lrtOracle.getRSETHPrice();
    rsethAmountToMint = (amount * assetPrice) / rsethPrice;
}
```

### Inlining Simple Checks

Inlining simple functions can save gas, especially if they are called frequently.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L32
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L169

**Instances:**
- Frequent calls to `UtilLib.checkNonZeroAddress`.

```solidity
UtilLib.checkNonZeroAddress(lrtConfigAddr);
```

**Proposed Changes:**
Inline the `checkNonZeroAddress` function if it is simple, for instance:

```solidity
require(lrtConfigAddr != address(0), "Zero address not allowed");
```

### Caching Results of External Functions

Caching results of external function calls can reduce gas costs, especially when the function is called multiple times within a transaction.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L44

**Instances:**
- Multiple calls to `lrtConfig.getContract` and `lrtConfig.assetStrategy`.

```solidity
address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);
address strategy = lrtConfig.assetStrategy(asset);
```

**Proposed Changes:**
Cache the results of these functions in local variables if they're used multiple times within the same function.

```solidity
function depositAssetIntoStrategy(address asset) external override {
    address strategy = lrtConfig.assetStrategy(asset); // Cached strategy
    IERC20 token = IERC20(asset);
    address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER); // Cached address

    uint256 balance = token.balanceOf(address(this));
    // Use the cached addresses...
}
```

### Inlining Simple Checks

Inlining simple functions can save gas, especially if they are frequently called.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L27

**Instances:**
- Calls to `UtilLib.checkNonZeroAddress`.

```solidity
UtilLib.checkNonZeroAddress(lrtConfigAddr);
```

**Proposed Changes:**
Inline the `checkNonZeroAddress` function if it is simple, for instance:

```solidity
require(lrtConfigAddr != address(0), "Zero address not allowed");
```

### Caching Results of External Functions

Caching the results of external function calls can significantly reduce gas costs, especially when these functions are called multiple times within the same transaction.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L52

**Instances:**
- Multiple calls to `getAssetPrice` within a loop in `getRSETHPrice`.

```solidity
for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
    address asset = supportedAssets[asset_idx];
    uint256 assetER = getAssetPrice(asset);
    // ...
}
```

**Proposed Changes:**
Cache the result of `getAssetPrice` outside the loop to avoid repeated external calls.

```solidity
function getRSETHPrice() external view returns (uint256 rsETHPrice) {
    // ...
    uint256[] memory assetPrices = new uint256[](supportedAssetCount);
    for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
        address asset = supportedAssets[asset_idx];
        assetPrices[asset_idx] = getAssetPrice(asset); // Cache the asset price here
        // ...
    }
    // Use assetPrices[asset_idx] in calculations
    // ...
}
```

### Inlining Simple Checks

Inlining simple validation checks can save gas compared to calling external libraries.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L30

**Instances:**
- Calls to `UtilLib.checkNonZeroAddress`.

```solidity
UtilLib.checkNonZeroAddress(lrtConfigAddr);
```

**Proposed Changes:**
Replace the call to `UtilLib.checkNonZeroAddress` with an inline require statement.

```solidity
require(lrtConfigAddr != address(0), "Zero address not allowed");
```

### Inlining Simple Checks

Inlining simple utility functions can save gas.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L33

**Instances:**
- Usage of `UtilLib.checkNonZeroAddress` for address validation.

```solidity
UtilLib.checkNonZeroAddress(admin);
UtilLib.checkNonZeroAddress(lrtConfigAddr);
```

**Proposed Changes:**
Inline these checks within the function to save gas:

```solidity
require(admin != address(0), "Zero address not allowed");
require(lrtConfigAddr != address(0), "Zero address not allowed");
```

### Inlining Simple Checks

Inlining simple utility functions can reduce gas usage.

**Instances:**
- Calls to `UtilLib.checkNonZeroAddress` for address validation.

```solidity
UtilLib.checkNonZeroAddress(lrtConfig_);
```

**Proposed Changes:**
Inline these checks to reduce gas costs:

```solidity
require(lrtConfig_ != address(0), "Zero address not allowed");
```

### Reducing Function Calls

Reducing unnecessary function calls within functions can save gas.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L46

**Instances:**
- `updatePriceFeedFor` checks the validity of `priceFeed` using `UtilLib.checkNonZeroAddress`.

```solidity
function updatePriceFeedFor(address asset, address priceFeed) external onlyLRTManager onlySupportedAsset(asset) {
    UtilLib.checkNonZeroAddress(priceFeed);
    // ...
}
```

**Proposed Changes:**
Inline the `checkNonZeroAddress` function:

```solidity
function updatePriceFeedFor(address asset, address priceFeed) external onlyLRTManager onlySupportedAsset(asset) {
    require(priceFeed != address(0), "Zero address not allowed");
    // ...
}
```

### Caching Frequently Accessed External Data

Caching results of external calls can be more efficient than making repeated calls.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L38

**Instances:**
- Access to the `assetPriceFeed` mapping in `getAssetPrice`.

```solidity
function getAssetPrice(address asset) external view onlySupportedAsset(asset) returns (uint256) {
    return AggregatorInterface(assetPriceFeed[asset]).latestAnswer();
}
```

**Proposed Changes:**
Cache the address from `assetPriceFeed` mapping in a local variable.

```solidity
function getAssetPrice(address asset) external view onlySupportedAsset(asset) returns (uint256) {
    address feed = assetPriceFeed[asset];
    return AggregatorInterface(feed).latestAnswer();
}
```

### Using Immutable for Fixed Values

Immutable variables are cheaper in terms of gas than regular state variables.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConfigRoleChecker.sol#L14

**Instances:**
- `lrtConfig` is a state variable that might be a candidate for `immutable` if it's set once and doesn't change.

```solidity
ILRTConfig public lrtConfig;
```

**Proposed Changes:**
If `lrtConfig` is set during initialization and does not change, consider making it immutable:

```solidity
ILRTConfig public immutable lrtConfig;
```

### Inlining Simple Checks

Inlining simple checks can save gas, especially if they are called frequently.

**Instances:**
- `UtilLib.checkNonZeroAddress` is used for address validation.

```solidity
UtilLib.checkNonZeroAddress(lrtConfigAddr);
```

**Proposed Changes:**
Inline this check to save gas:

```solidity
require(lrtConfigAddr != address(0), "Zero address not allowed");
```

### Minimizing Role Checks

The `onlyLRTManager` and `onlyLRTAdmin` modifiers use `IAccessControl` for role verification every time they're called, leading to repeated external calls to the `lrtConfig` contract. This can be inefficient, especially if multiple role checks are performed within a single transaction.


**Instances:**


  ```solidity
  modifier onlyLRTManager() {
      if (!IAccessControl(address(lrtConfig)).hasRole(LRTConstants.MANAGER, msg.sender)) {
          revert ILRTConfig.CallerNotLRTConfigManager();
      }
      _;
  }
  ```

**Proposed Changes:**

To optimize gas usage, consider caching the role in a local variable when the same check is performed multiple times within a single transaction. This approach reduces the number of external calls, especially in functions that utilize these modifiers multiple times.

  ```solidity
  modifier onlyLRTManager() {
      // Cache role check result
      bool isManager = IAccessControl(address(lrtConfig)).hasRole(LRTConstants.MANAGER, msg.sender);
      if (!isManager) {
          revert ILRTConfig.CallerNotLRTConfigManager();
      }
      _;
  }
  ```