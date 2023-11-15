## Summary

### Low Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| [[L&#x2011;01](#l01-tokenMap-mapping)] | The `tokenMap` mapping in the `LRTConfig` contract serves no significant purpose in the protocol, and its presence does not impact the current implementation. | 1 |
| [[L&#x2011;02](#l02-three-mappings)] | The three mappings in the `LRTConfig` contract can be consolidated into a single mapping using a struct. | 1 |
| [[L&#x2011;03](#l03-missing-events)] | The `tokenMap` mapping in the `LRTConfig` contract serves no significant purpose in the protocol, and its presence does not impact the current implementation. | 1 |
| [[L&#x2011;04](#l04-missing-events)] | Missing Event for critical functions or parameters change | 2 |
| [[L&#x2011;05](#l05-restricting-updating-rsETH)] | Consider restricting updating rsETH contract address after the initial deployment in `LRTConfig` contract | 1 |
| [[L&#x2011;06](#l06-prevent-underflow-errors)] | Prevent underflow errors when depositing assets after reducing the deposit limit for assets. | 1 |
| [[L&#x2011;07](#l07-verify-the-value )] | Verify the value of the `amount` parameter to prevent unnecessary gas losses. | 2 |
| [[L&#x2011;08](#l08-including-the-depositor)] | Consider including the depositor's (msg.sender) address in the `AssetDeposit` event. | 1 |
| [[L&#x2011;09](#l09-functionality-is-not-utilized)] | The `Pause/Unpause` functionality is not utilized in `src/LRTOracle.sol`. | 1 |

### Non-critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [[N&#x2011;01](#n01-relocating-the-definition)] | Consider relocating the definition of `DEFAULT_ADMIN_ROLE` to `LRTConstants.sol` for better adherence to coding standards. | 1 |
| [[N&#x2011;02](#n02-verify-the-current-value)] | Verify the current value before assigning the new value to prevent unnecessary gas consumption in the event of setting the same value. | 4 |
| [[N&#x2011;03](#n03-renaming-the-function)] | Consider renaming the function `updateAssetStrategy()` to `setAssetStrategy()` for improved code standardization. | 1 |
| [[N&#x2011;04](#n04-remove-comment)] | Remove unwanted comments | 1 |


## Low Risk Issues


### [L&#x2011;01] The `tokenMap` mapping in the `LRTConfig` contract serves no significant purpose in the protocol, and its presence does not impact the current implementation.

The functions `getLSTToken()`, `setToken()`, `_setToken()`, and the `tokenMap` in the protocol currently have no practical application. The addition of a `token` to `tokenMap` is unused, and the protocol operates effectively without these functionalities. While these values may be relevant for future extensions, their current significance is minimal.

*There is 1 instance of this issue:*

```solidity
File: src/LRTConfig.sol

13:    mapping(bytes32 tokenKey => address tokenAddress) public tokenMap;

58:        _setToken(LRTConstants.ST_ETH_TOKEN, stETH);
59:        _setToken(LRTConstants.R_ETH_TOKEN, rETH);
60:        _setToken(LRTConstants.CB_ETH_TOKEN, cbETH);

127:    function getLSTToken(bytes32 tokenKey) external view override returns (address) {
128:        return tokenMap[tokenKey];

149:    function setToken(bytes32 tokenKey, address assetAddress) external onlyRole(DEFAULT_ADMIN_ROLE) {
150:        _setToken(tokenKey, assetAddress);

156:    function _setToken(bytes32 key, address val) private {
157:        UtilLib.checkNonZeroAddress(val);
158:        if (tokenMap[key] == val) {
159:            revert ValueAlreadyInUse();
160:        }
161:        tokenMap[key] = val;

```
*GitHub*: [13](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L13)

### [L&#x2011;02] The three mappings in the `LRTConfig` contract can be consolidated into a single mapping using a struct.

Currently, the contract uses three separate mappings to store information about each asset: `isSupportedAsset`, `depositLimitByAsset`, and `assetStrategy`. This approach can be improved for better readability, efficiency, and maintainability by consolidating these mappings into a single mapping that uses a struct to store asset information.

*There is 1 instance of this issue:*

```solidity
File: src/LRTConfig.sol

15:    mapping(address token => bool isSupported) public isSupportedAsset;
16:    mapping(address token => uint256 amount) public depositLimitByAsset;
17:    mapping(address token => address strategy) public override assetStrategy;
```
*GitHub*: [15](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L15C4-L17C78)

Proposed Change:

```solidity
struct AssetInfo {
    bool isSupported;
    uint256 depositLimit;
    address strategy;
}

mapping(address => AssetInfo) public assets;
```

Benefits:

1. Readability: Consolidating related data into a single struct improves code readability and understanding.
2. Efficiency: Accessing a struct in a mapping requires only one lookup, making it more gas efficient compared to multiple lookups in separate mappings.
3. Maintainability: Future modifications become easier as adding a new property to the asset only requires adding it to the struct, instead of creating a new mapping.
4. Organization: Structs logically group related data together, which can help prevent bugs and make the code easier to reason about.

This change should be implemented carefully, ensuring all instances where these properties are accessed or modified are updated accordingly.

### [L&#x2011;03] The `tokenMap` mapping in the `LRTConfig` contract serves no significant purpose in the protocol, and its presence does not impact the current implementation.

The functions `getLSTToken()`, `setToken()`, `_setToken()`, and the `tokenMap` in the protocol currently have no practical application. The addition of a `token` to `tokenMap` is unused, and the protocol operates effectively without these functionalities. While these values may be relevant for future extensions, their current significance is minimal.

*There is 1 instance of this issue:*

```solidity
File: src/LRTConfig.sol

13:    mapping(bytes32 tokenKey => address tokenAddress) public tokenMap;

58:        _setToken(LRTConstants.ST_ETH_TOKEN, stETH);
59:        _setToken(LRTConstants.R_ETH_TOKEN, rETH);
60:        _setToken(LRTConstants.CB_ETH_TOKEN, cbETH);

127:    function getLSTToken(bytes32 tokenKey) external view override returns (address) {
128:        return tokenMap[tokenKey];

149:    function setToken(bytes32 tokenKey, address assetAddress) external onlyRole(DEFAULT_ADMIN_ROLE) {
150:        _setToken(tokenKey, assetAddress);

156:    function _setToken(bytes32 key, address val) private {
157:        UtilLib.checkNonZeroAddress(val);
158:        if (tokenMap[key] == val) {
159:            revert ValueAlreadyInUse();
160:        }
161:        tokenMap[key] = val;

```
*GitHub*: [13](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L13)

### [L&#x2011;04] Missing Event for critical functions or parameters change

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

*There is 2 instance of this issue:*

```solidity
File: src/LRTConfig.sol

109:    function updateAssetStrategy(

144:    function setRSETH(address rsETH_) external onlyRole(DEFAULT_ADMIN_ROLE) {

```

*GitHub*: [109](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L109), [144](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L144)

### [L&#x2011;05] Consider restricting updating rsETH contract address after the initial deployment in `LRTConfig` contract

rsETH contract is an upgradable contract and the is not going to be changed in future, so consider restricting further updating rsETH to prevent accidental updates.

*There is 1 instance of this issue:*

```solidity
File: src/LRTConfig.sol

144:    function setRSETH(address rsETH_) external onlyRole(DEFAULT_ADMIN_ROLE) {
145:        UtilLib.checkNonZeroAddress(rsETH_);
146:            rsETH = rsETH_;

```
*GitHub*: [144](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L144C4-L146C24)

### [L&#x2011;06] Prevent underflow errors when depositing assets after reducing the deposit limit for assets.

The `LRTDepositPool` contract does not currently handle underflow errors when depositing assets after reducing the deposit limit for a specific asset.

*There is 1 instance of this issue:*

```solidity
File: src/LRTDepositPool.sol

56:    function getAssetCurrentLimit(address asset) public view override returns (uint256) {
57:        return lrtConfig.depositLimitByAsset(asset) - getTotalAssetDeposits(asset);
58:    }

```
*GitHub*: [56](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L56C1-L58C6)

The expression `lrtConfig.depositLimitByAsset(asset)` on line number 56 always assumes a value greater than or equal to the `getTotalAssetDeposits(asset)` value. However, if the protocol reduces the deposit limit of an asset after some deposits have been made, it could result in `lrtConfig.depositLimitByAsset(asset)` being less than `getTotalAssetDeposits(asset)`. In such cases, instead of triggering the expected `MaximumDepositLimitReached()` error, an underflow error may occur.

### [L&#x2011;07] Verify the value of the `amount` parameter to prevent unnecessary gas losses.

Verify the value of the `amount` parameter to avoid unnecessary gas losses by preventing zero-amount transfers.

*There is 2 instance of this issue:*

```solidity
File: src/LRTDepositPool.sol

183:    function transferAssetToNodeDelegator(
184:        uint256 ndcIndex,
185:        address asset,
186:        uint256 amount
187:    )
188:        external
189:        nonReentrant
190:        onlyLRTManager
191:        onlySupportedAsset(asset)
192:    {
193:        address nodeDelegator = nodeDelegatorQueue[ndcIndex];
194:  @>    if (!IERC20(asset).transfer(nodeDelegator, amount)) {
195:            revert TokenTransferFailed();

```
*GitHub*: [183](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L183C4-L195C42)

```solidity
File: src/NodeDelegator.sol

74:    function transferBackToLRTDepositPool(
75:        address asset,
76:        uint256 amount
77:    )
78:        external
79:        whenNotPaused
80:        nonReentrant
81:        onlySupportedAsset(asset)
82:        onlyLRTManager
83:    {
84:        address lrtDepositPool = lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL);

86:  @>    if (!IERC20(asset).transfer(lrtDepositPool, amount)) {

```
*GitHub*: [74](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L74C2-L86C63)


### [L&#x2011;08] Consider including the depositor's (msg.sender) address in the `AssetDeposit` event.

Consider including the depositor's (msg.sender) address in the `AssetDeposit` event to provide clarity on the depositor's identity. This addition is particularly important for off-chain systems heavily reliant on events to avoid confusion.

*There is 1 instance of this issue:*

```solidity
File: src/LRTDepositPool.sol

143:        emit AssetDeposit(asset, depositAmount, rsethAmountMinted);

```
*GitHub*: [143](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L143)

### [L&#x2011;09] The `Pause/Unpause` functionality is not utilized in `src/LRTOracle.sol`.

The `pause()` and `unpause()` functions are present in the `LRTOracle` contract; however, they currently have no impact on the contract's behavior. Despite invoking these functions, the contract's functions remain accessible whether the contract is paused or unpaused.

*There is 1 instance of this issue:*

```solidity
File: src/LRTOracle.sol

102:    function pause() external onlyLRTManager {
103:        _pause();
104:    }

107:    function unpause() external onlyLRTAdmin {
108:        _unpause();
109:    }

```
*GitHub*: [102](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L102C1-L109C6)


## Non-critical Issues

### [N&#x2011;01] Consider relocating the definition of `DEFAULT_ADMIN_ROLE` to `LRTConstants.sol` for better adherence to coding standards.

The definition of `DEFAULT_ADMIN_ROLE` inside `LRTConfigRoleChecker#onlyLRTAdmin()` could be moved to `LRTConstants.sol` to enhance adherence to coding standards.

*There are 1 instances of this issue:*

```solidity
File: src/utils/LRTConfigRoleChecker.sol

27:    modifier onlyLRTAdmin() {
28: @>     bytes32 DEFAULT_ADMIN_ROLE = 0x00;

```
*GitHub*: [28](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConfigRoleChecker.sol#L28C14-L28C14)

### [N&#x2011;02] Verify the current value before assigning the new value to prevent unnecessary gas consumption in the event of setting the same value.

*There are 4 instances of this issue:*

```solidity
File: src/LRTConfig.sol

102        depositLimitByAsset[asset] = depositLimit;

144:        rsETH = rsETH_;

```
*GitHub*: [102](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L102), [144](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L144)

```solidity
File: src/LRTDepositPool.sol

203:        maxNodeDelegatorCount = maxNodeDelegatorCount_;
```

*GitHub*: [203](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L203)

```solidity
File: src/LRTOracle.sol

97:        assetPriceOracle[asset] = priceOracle;
```
*GitHub*: [97](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L97)

```solidity
File: src/oracles/ChainlinkPriceOracle.sol

47:        assetPriceFeed[asset] = priceFeed;
```
*GitHub*: [47](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/oracles/ChainlinkPriceOracle.sol#L47)

### [N&#x2011;03] Consider renaming the function `updateAssetStrategy()` to `setAssetStrategy()` for improved code standardization.

There is no distinct function solely dedicated to adding a strategy. The existing `updateAssetStrategy()` function serves both the purpose of adding new strategies and updating existing ones. To enhance code standardization, it would be advisable to rename `updateAssetStrategy()` as `setAssetStrategy()`.

*There are 1 instances of this issue:*

```solidity
File: src/LRTConfig.sol

109:    function updateAssetStrategy(

```
*GitHub*: [109](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L109)

### [N&#x2011;04] Remove unwanted comments

*There are 1 instances of this issue:*

```solidity
File: src/LRTDepositPool.sol

78:        // Question: is here the right place to have this? Could it be in LRTConfig?

```
*GitHub*: [78](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L78)
