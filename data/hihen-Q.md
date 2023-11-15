# QA Report

## Summary

### Low Issues

Total **23 instances** over **4 issues**:

|ID|Issue|Instances|
|:--:|:---|:--:|
| [[L-01]](#l-01-owner-can-renounce-while-system-is-paused) | Owner can renounce while system is paused | 4 |
| [[L-02]](#l-02-revert-on-transfer-to-the-zero-address) | Revert on transfer to the zero address | 2 |
| [[L-03]](#l-03-critical-functions-should-be-controlled-by-time-locks) | Critical functions should be controlled by time locks | 14 |
| [[L-04]](#l-04-contracts-are-vulnerable-to-rebasing-accounting-related-issues) | Contracts are vulnerable to rebasing accounting-related issues | 3 |

### Non Critical Issues

Total **40 instances** over **8 issues**:

|ID|Issue|Instances|
|:--:|:---|:--:|
| [[N-01]](#n-01-redundant-inheritance-specifier) | Redundant inheritance specifier | 1 |
| [[N-02]](#n-02-events-should-be-emitted-before-external-calls) | Events should be emitted before external calls | 1 |
| [[N-03]](#n-03-openzeppelincontracts-should-be-upgraded-to-a-newer-version) | @openzeppelin/contracts should be upgraded to a newer version | 11 |
| [[N-04]](#n-04-expressions-for-constant-values-should-use-immutable-rather-than-constant) | Expressions for constant values should use `immutable` rather than `constant` | 9 |
| [[N-05]](#n-05-unnecessary-casting) | Unnecessary casting | 2 |
| [[N-06]](#n-06-using-underscore-at-the-end-of-variable-name) | Using underscore at the end of variable name | 5 |
| [[N-07]](#n-07-multiple-mappings-with-same-keys-can-be-combined-into-a-single-struct-mapping-for-readability) | Multiple mappings with same keys can be combined into a single struct mapping for readability | 5 |
| [[N-08]](#n-08-inconsistent-spacing-in-comments) | Inconsistent spacing in comments | 6 |

## Low Issues

### [L-01] Owner can renounce while system is paused

The contract owner or single user with a role is not prevented from renouncing the role/ownership while the contract is paused, which would cause any user assets stored in the protocol, to be locked indefinitely.

<details>
<summary>There are 4 instances (click to show):</summary>

- *LRTDepositPool.sol* ( [208-210](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L208-L210) ):

```solidity
208:     function pause() external onlyLRTManager {
209:         _pause();
210:     }
```

- *LRTOracle.sol* ( [102-104](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L102-L104) ):

```solidity
102:     function pause() external onlyLRTManager {
103:         _pause();
104:     }
```

- *NodeDelegator.sol* ( [127-129](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L127-L129) ):

```solidity
127:     function pause() external onlyLRTManager {
128:         _pause();
129:     }
```

- *RSETH.sol* ( [60-62](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/RSETH.sol#L60-L62) ):

```solidity
60:     function pause() external onlyLRTManager {
61:         _pause();
62:     }
```

</details>

### [L-02] Revert on transfer to the zero address

It's good practice to revert a token transfer transaction if the recipient's address is the zero address. This can prevent unintentional transfers to the zero address due to accidental operations or programming errors. Many token contracts implement such a safeguard, such as [OpenZeppelin - ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/9e3f4d60c581010c4a3979480e07cc7752f124cc/contracts/token/ERC20/ERC20.sol#L232), [OpenZeppelin - ERC721](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/9e3f4d60c581010c4a3979480e07cc7752f124cc/contracts/token/ERC721/ERC721.sol#L142).

There are 2 instances:

- *LRTDepositPool.sol* ( [194](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L194) ):

```solidity
194:         if (!IERC20(asset).transfer(nodeDelegator, amount)) {
```

- *NodeDelegator.sol* ( [86](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L86) ):

```solidity
86:         if (!IERC20(asset).transfer(lrtDepositPool, amount)) {
```

### [L-03] Critical functions should be controlled by time locks

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

<details>
<summary>There are 14 instances (click to show):</summary>

- *LRTConfig.sol* ( [73](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L73), [94-101](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L94-L101), [109-116](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L109-L116), [144](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L144), [149](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L149), [165](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L165) ):

```solidity
73:     function addNewSupportedAsset(address asset, uint256 depositLimit) external onlyRole(LRTConstants.MANAGER) {

94:     function updateAssetDepositLimit(
95:         address asset,
96:         uint256 depositLimit
97:     )
98:         external
99:         onlyRole(LRTConstants.MANAGER)
100:         onlySupportedAsset(asset)
101:     {

109:     function updateAssetStrategy(
110:         address asset,
111:         address strategy
112:     )
113:         external
114:         onlyRole(DEFAULT_ADMIN_ROLE)
115:         onlySupportedAsset(asset)
116:     {

144:     function setRSETH(address rsETH_) external onlyRole(DEFAULT_ADMIN_ROLE) {

149:     function setToken(bytes32 tokenKey, address assetAddress) external onlyRole(DEFAULT_ADMIN_ROLE) {

165:     function setContract(bytes32 contractKey, address contractAddress) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

- *LRTDepositPool.sol* ( [162](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L162), [202](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L202), [208](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L208) ):

```solidity
162:     function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {

202:     function updateMaxNodeDelegatorCount(uint256 maxNodeDelegatorCount_) external onlyLRTAdmin {

208:     function pause() external onlyLRTManager {
```

- *LRTOracle.sol* ( [88-95](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L88-L95) ):

```solidity
88:     function updatePriceOracleFor(
89:         address asset,
90:         address priceOracle
91:     )
92:         external
93:         onlyLRTManager
94:         onlySupportedAsset(asset)
95:     {
```

- *NodeDelegator.sol* ( [127](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L127) ):

```solidity
127:     function pause() external onlyLRTManager {
```

- *RSETH.sol* ( [73](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/RSETH.sol#L73) ):

```solidity
73:     function updateLRTConfig(address _lrtConfig) external override onlyRole(DEFAULT_ADMIN_ROLE) {
```

- *ChainlinkPriceOracle.sol* ( [45](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/oracles/ChainlinkPriceOracle.sol#L45) ):

```solidity
45:     function updatePriceFeedFor(address asset, address priceFeed) external onlyLRTManager onlySupportedAsset(asset) {
```

- *LRTConfigRoleChecker.sol* ( [47](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConfigRoleChecker.sol#L47) ):

```solidity
47:     function updateLRTConfig(address lrtConfigAddr) external virtual onlyLRTAdmin {
```

</details>

### [L-04] Contracts are vulnerable to rebasing accounting-related issues

Rebasing tokens are tokens that have each holder's `balanceof()` increase over time. Aave aTokens are an example of such tokens. If rebasing tokens are used, rewards accrue to the contract holding the tokens, and cannot be withdrawn by the original depositor. To address the issue, track 'shares' deposited on a pro-rata basis, and let shares be redeemed for their proportion of the current balance at the time of the withdrawal.

<details>
<summary>There are 3 instances (click to show):</summary>

- *LRTDepositPool.sol* ( [119-127](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L119-L127), [183-192](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L183-L192) ):

```solidity
119:     function depositAsset(
120:         address asset,
121:         uint256 depositAmount
122:     )
123:         external
124:         whenNotPaused
125:         nonReentrant
126:         onlySupportedAsset(asset)
127:     {

183:     function transferAssetToNodeDelegator(
184:         uint256 ndcIndex,
185:         address asset,
186:         uint256 amount
187:     )
188:         external
189:         nonReentrant
190:         onlyLRTManager
191:         onlySupportedAsset(asset)
192:     {
```

- *NodeDelegator.sol* ( [74-83](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L74-L83) ):

```solidity
74:     function transferBackToLRTDepositPool(
75:         address asset,
76:         uint256 amount
77:     )
78:         external
79:         whenNotPaused
80:         nonReentrant
81:         onlySupportedAsset(asset)
82:         onlyLRTManager
83:     {
```

</details>

## Non Critical Issues

### [N-01] Redundant inheritance specifier

The contracts below already extend the specified contract, so there is no need to list it in the inheritance list again.

There is 1 instance:

- *RSETH.sol* ( [14-19](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/RSETH.sol#L14-L19) ):

```solidity
/// @audit ERC20Upgradeable already extends Initializable
14: contract RSETH is
15:     Initializable,
16:     LRTConfigRoleChecker,
17:     ERC20Upgradeable,
18:     PausableUpgradeable,
19:     AccessControlUpgradeable
```

### [N-02] Events should be emitted before external calls

Ensure that events follow the best practice of check-effects-interaction, and are emitted before external calls.

There is 1 instance:

- *LRTDepositPool.sol* ( [143](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L143) ):

```solidity
/// @audit `_mintRsETH()` is called on line 141, triggering an external call - `mint()`
143:         emit AssetDeposit(asset, depositAmount, rsethAmountMinted);
```

### [N-03] @openzeppelin/contracts should be upgraded to a newer version

These contracts import contracts from `@openzeppelin/contracts`, but they are not using [the latest version](https://github.com/OpenZeppelin/openzeppelin-contracts/releases).

The imported version is `4.9.3`.

<details>
<summary>There are 11 instances (click to show):</summary>

- *LRTConfig.sol* ( [8](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L8) ):

```solidity
8: import { AccessControlUpgradeable } from "@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol";
```

- *LRTDepositPool.sol* ( [13](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L13), [13](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L13) ):

```solidity
13: import { IERC20 } from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

13: import { IERC20 } from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```

- *LRTOracle.sol* ( [14](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L14), [14](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L14) ):

```solidity
14: import { IERC20 } from "@openzeppelin/contracts/interfaces/IERC20.sol";

14: import { IERC20 } from "@openzeppelin/contracts/interfaces/IERC20.sol";
```

- *NodeDelegator.sol* ( [12](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L12), [12](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L12) ):

```solidity
12: import { IERC20 } from "@openzeppelin/contracts/interfaces/IERC20.sol";

12: import { IERC20 } from "@openzeppelin/contracts/interfaces/IERC20.sol";
```

- *RSETH.sol* ( [7](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/RSETH.sol#L7) ):

```solidity
7: import { ERC20Upgradeable, Initializable } from "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
```

- *ChainlinkPriceOracle.sol* ( [9](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/oracles/ChainlinkPriceOracle.sol#L9) ):

```solidity
9: import { Initializable } from "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
```

- *LRTConfigRoleChecker.sol* ( [9](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConfigRoleChecker.sol#L9), [9](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConfigRoleChecker.sol#L9) ):

```solidity
9: import { IAccessControl } from "@openzeppelin/contracts/access/IAccessControl.sol";

9: import { IAccessControl } from "@openzeppelin/contracts/access/IAccessControl.sol";
```

</details>

### [N-04] Expressions for constant values should use `immutable` rather than `constant`

While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use the right tool for the task at hand. There is a difference between `constant` variables and `immutable` variables, and they should each be used in their appropriate contexts. `constants` should be used for literal values written into the code, and `immutable` variables should be used for expressions, or values calculated in, or passed into the constructor.

There are 9 instances:

- *RSETH.sol* ( [21](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/RSETH.sol#L21), [22](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/RSETH.sol#L22) ):

```solidity
21:     bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

22:     bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

- *LRTConstants.sol* ( [7](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConstants.sol#L7), [9](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConstants.sol#L9), [11](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConstants.sol#L11), [14](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConstants.sol#L14), [15](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConstants.sol#L15), [16](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConstants.sol#L16), [19](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConstants.sol#L19) ):

```solidity
7:     bytes32 public constant R_ETH_TOKEN = keccak256("R_ETH_TOKEN");

9:     bytes32 public constant ST_ETH_TOKEN = keccak256("ST_ETH_TOKEN");

11:     bytes32 public constant CB_ETH_TOKEN = keccak256("CB_ETH_TOKEN");

14:     bytes32 public constant LRT_ORACLE = keccak256("LRT_ORACLE");

15:     bytes32 public constant LRT_DEPOSIT_POOL = keccak256("LRT_DEPOSIT_POOL");

16:     bytes32 public constant EIGEN_STRATEGY_MANAGER = keccak256("EIGEN_STRATEGY_MANAGER");

19:     bytes32 public constant MANAGER = keccak256("MANAGER");
```

### [N-05] Unnecessary casting

Unnecessary castings can be removed.

There are 2 instances:

- *NodeDelegator.sol* ( [110](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L110), [111](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L111) ):

```solidity
/// @audit IStrategy(strategies[i])
110:             assets[i] = address(IStrategy(strategies[i]).underlyingToken());

/// @audit IStrategy(strategies[i])
111:             assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
```

### [N-06] Using underscore at the end of variable name

The use of the underscore at the end of the variable name is unusual, consider refactoring it.

There are 5 instances:

- *LRTConfig.sol* ( [46](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L46), [144](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L144) ):

```solidity
/// @audit rsETH_
46:         address rsETH_

/// @audit rsETH_
144:     function setRSETH(address rsETH_) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

- *LRTDepositPool.sol* ( [202](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L202) ):

```solidity
/// @audit maxNodeDelegatorCount_
202:     function updateMaxNodeDelegatorCount(uint256 maxNodeDelegatorCount_) external onlyLRTAdmin {
```

- *ChainlinkPriceOracle.sol* ( [27](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/oracles/ChainlinkPriceOracle.sol#L27) ):

```solidity
/// @audit lrtConfig_
27:     function initialize(address lrtConfig_) external initializer {
```

- *UtilLib.sol* ( [11](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/UtilLib.sol#L11) ):

```solidity
/// @audit address_
11:     function checkNonZeroAddress(address address_) internal pure {
```

### [N-07] Multiple mappings with same keys can be combined into a single struct mapping for readability

Well-organized data structures make code reviews easier, which may lead to fewer bugs. Consider combining related mappings into mappings to structs, so it's clear what data is related.

There are 5 instances:

- *LRTConfig.sol* ( [13](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L13), [14](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L14), [15](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L15), [16](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L16), [17](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L17) ):

```solidity
13:     mapping(bytes32 tokenKey => address tokenAddress) public tokenMap;

14:     mapping(bytes32 contractKey => address contractAddress) public contractMap;

15:     mapping(address token => bool isSupported) public isSupportedAsset;

16:     mapping(address token => uint256 amount) public depositLimitByAsset;

17:     mapping(address token => address strategy) public override assetStrategy;
```

### [N-08] Inconsistent spacing in comments

The comments below are in the `//x` format, which differs from the commonly used idiomatic comment syntax of `//<space>x`. It is recommended to use a consistent comment syntax throughout.

There are 6 instances:

- *LRTConstants.sol* ( [5](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConstants.sol#L5), [6](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConstants.sol#L6), [8](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConstants.sol#L8), [10](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConstants.sol#L10), [13](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConstants.sol#L13), [18](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConstants.sol#L18) ):

```solidity
5:     //tokens

6:     //rETH token

8:     //stETH token

10:     //cbETH token

13:     //contracts

18:     //Roles
```
