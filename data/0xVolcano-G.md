# Gas report

## Table of Contents

- [Gas report](#gas-report)
  - [Table of Contents](#table-of-contents)
  - [Avoid zero to none zero storage writes](#avoid-zero-to-none-zero-storage-writes)
  - [We can optimize the function `updateLRTConfig()`](#we-can-optimize-the-function-updatelrtconfig)
  - [Emit local variables instead of state variables(not found by bot)](#emit-local-variables-instead-of-state-variablesnot-found-by-bot)
  - [Don't cache calls/variables  if using them once](#dont-cache-callsvariables--if-using-them-once)
    - [No need to cache `lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER)`](#no-need-to-cache-lrtconfiggetcontractlrtconstantseigen_strategy_manager)
    - [No need to cache `lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER)`](#no-need-to-cache-lrtconfiggetcontractlrtconstantseigen_strategy_manager-1)
    - [No need to cache `lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL)`](#no-need-to-cache-lrtconfiggetcontractlrtconstantslrt_deposit_pool)
    - [No need to cache `lrtConfig.assetStrategy(asset)`](#no-need-to-cache-lrtconfigassetstrategyasset)
    - [No need to cache `nodeDelegatorQueue[ndcIndex]`](#no-need-to-cache-nodedelegatorqueuendcindex)
  - [Caching the length of calldata increases gas cost instead of reducing it](#caching-the-length-of-calldata-increases-gas-cost-instead-of-reducing-it)


## Avoid zero to none zero storage writes

When a storage variable goes from zero to non-zero, the user must pay 22,100 gas total (20,000 gas for a zero to non-zero write and 2,100 for a cold storage access).

This is why the Openzeppelin reentrancy guard registers functions as active or not with 1 and 2 rather than 0 and 1. It only costs 5,000 gas to alter a storage variable from non-zero to non-zero.

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L19-L38
```solidity
File: /src/LRTDepositPool.sol
19:contract LRTDepositPool is ILRTDepositPool, LRTConfigRoleChecker, PausableUpgradeable, ReentrancyGuardUpgradeable {
20:    uint256 public maxNodeDelegatorCount;

29:    /// @dev Initializes the contract
30:    /// @param lrtConfigAddr LRT config address
31:    function initialize(address lrtConfigAddr) external initializer {
32:        UtilLib.checkNonZeroAddress(lrtConfigAddr);
33:        __Pausable_init();
34:        __ReentrancyGuard_init();
35:        maxNodeDelegatorCount = 10;
36:        lrtConfig = ILRTConfig(lrtConfigAddr);
37:        emit UpdatedLRTConfig(lrtConfigAddr);
38:    }
```

When declaring the variable `maxNodDelegatorCount` the default value is applied since we do not assign anything, the default value of `uint256` is `0`
We then call the function `initialize()` and here we assign the value `10` `maxNodeDelegatorCount = 10;`. This means our value is updated from `0`  to `10`

Since the `initialize()` function will always be called first, we can initialize our variable `maxNodDelegatorCount`  with `1` during declaration which should avoid the zero to non zero writes

```diff
 /// @title LRTDepositPool - Deposit Pool Contract for LSTs
 /// @notice Handles LST asset deposits
 contract LRTDepositPool is ILRTDepositPool, LRTConfigRoleChecker, PausableUpgradeable, ReentrancyGuardUpgradeable {
-    uint256 public maxNodeDelegatorCount;
+    uint256 public maxNodeDelegatorCount = 1;

     address[] public nodeDelegatorQueue;

```

## We can optimize the function `updateLRTConfig()`

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConfigRoleChecker.sol#L47-L51
```solidity
File: /src/utils/LRTConfigRoleChecker.sol
47:    function updateLRTConfig(address lrtConfigAddr) external virtual onlyLRTAdmin {
48:        UtilLib.checkNonZeroAddress(lrtConfigAddr);
49:        lrtConfig = ILRTConfig(lrtConfigAddr);
50:        emit UpdatedLRTConfig(lrtConfigAddr);
51:    }
```

The above function uses the modifier `onlyLRTAdmin` which has the following implementation
```solidity
    modifier onlyLRTAdmin() {
        bytes32 DEFAULT_ADMIN_ROLE = 0x00;
        if (!IAccessControl(address(lrtConfig)).hasRole(DEFAULT_ADMIN_ROLE, msg.sender)) {
            revert ILRTConfig.CallerNotLRTConfigAdmin();
        }
        _;
    }
```

The modifier reads a state variable `lrtConfig` and also makes an external call to `hasRole()`

If the check does not pass we have a revert. Using this modifier in our function means we run the check first and revert if the check does not pass.
If the check passes, our function has one more check `UtilLib.checkNonZeroAddress` which basically checks that our address is not zero. This is way cheaper compared to the check inside the modifier

If this check does not pass, it means we would also revert, now suppose the first check(modifier) passes but the non zero fails, the gas consumed inside the modifier doing the state read and external call would all be wasted.
We can do this checks more efficiently by prioritizing cheaper checks first but for this to work, we would need to inline our modifier as shown below


```diff
diff --git a/src/utils/LRTConfigRoleChecker.sol b/src/utils/LRTConfigRoleChecker.sol
index 991d0e9..1b9917f 100644
--- a/src/utils/LRTConfigRoleChecker.sol
+++ b/src/utils/LRTConfigRoleChecker.sol
@@ -44,8 +44,12 @@ abstract contract LRTConfigRoleChecker {
     /// @notice Updates the LRT config contract
     /// @dev only callable by LRT admin
     /// @param lrtConfigAddr the new LRT config contract Address
-    function updateLRTConfig(address lrtConfigAddr) external virtual onlyLRTAdmin {
+    function updateLRTConfig(address lrtConfigAddr) external virtual  {
         UtilLib.checkNonZeroAddress(lrtConfigAddr);
+        bytes32 DEFAULT_ADMIN_ROLE = 0x00;
+        if (!IAccessControl(address(lrtConfig)).hasRole(DEFAULT_ADMIN_ROLE, msg.sender)) {
+            revert ILRTConfig.CallerNotLRTConfigAdmin();
+        }
         lrtConfig = ILRTConfig(lrtConfigAddr);
         emit UpdatedLRTConfig(lrtConfigAddr);
     }
     
```


## Emit local variables instead of state variables(not found by bot)

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L202-L205
```solidity
File: /src/LRTDepositPool.sol
202:    function updateMaxNodeDelegatorCount(uint256 maxNodeDelegatorCount_) external onlyLRTAdmin {
203:        maxNodeDelegatorCount = maxNodeDelegatorCount_;
204:        emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount);
205:    }
```

```diff
@@ -201,7 +201,7 @@ contract LRTDepositPool is ILRTDepositPool, LRTConfigRoleChecker, PausableUpgrad
     /// @param maxNodeDelegatorCount_ Maximum count of node delegator
     function updateMaxNodeDelegatorCount(uint256 maxNodeDelegatorCount_) external onlyLRTAdmin {
         maxNodeDelegatorCount = maxNodeDelegatorCount_;
-        emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount);
+        emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount_);//@audit emit the local var
     }
```

## Don't cache calls/variables  if using them once

### No need to cache `lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER)`

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L38-L46
```solidity
File: /src/NodeDelegator.sol
38:    function maxApproveToEigenStrategyManager(address asset)
39:        external
40:        override
41:        onlySupportedAsset(asset)
42:        onlyLRTManager
43:    {
44:        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);
45:        IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);
46:    }
```

The variable `eigenlayerStrategyManagerAddress` is only used once, therefore no need to cache the call

```diff
diff --git a/src/NodeDelegator.sol b/src/NodeDelegator.sol
index bba20ca..989218c 100644
--- a/src/NodeDelegator.sol
+++ b/src/NodeDelegator.sol
@@ -41,8 +41,7 @@ contract NodeDelegator is INodeDelegator, LRTConfigRoleChecker, PausableUpgradea
         onlySupportedAsset(asset)
         onlyLRTManager
     {
-        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);
-        IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);
+        IERC20(asset).approve(lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY
```

### No need to cache `lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER)`

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L51-L68

```solidity
File: /src/NodeDelegator.sol
51:    function depositAssetIntoStrategy(address asset)

58:    {
59:        address strategy = lrtConfig.assetStrategy(asset);
60:        IERC20 token = IERC20(asset);
61:        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

63:        uint256 balance = token.balanceOf(address(this));

65:        emit AssetDepositIntoStrategy(asset, strategy, balance);

67:IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
68:    }
```

```diff
     /// @notice Deposits an asset lying in this NDC into its strategy
@@ -58,13 +58,12 @@ contract NodeDelegator is INodeDelegator, LRTConfigRoleChecker, PausableUpgradea
     {
         address strategy = lrtConfig.assetStrategy(asset);
         IERC20 token = IERC20(asset);
-        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

         uint256 balance = token.balanceOf(address(this));

         emit AssetDepositIntoStrategy(asset, strategy, balance);

-        IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
+        IEigenStrategyManager(lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER)).depositIntoStrategy(IStrategy(strategy), token, balance);
     }

```

### No need to cache `lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL)`

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L74-L89


```solidity
File: /src/NodeDelegator.sol
74:    function transferBackToLRTDepositPool(

84:        address lrtDepositPool = lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL);

86:        if (!IERC20(asset).transfer(lrtDepositPool, amount)) {
87:            revert TokenTransferFailed();
88:        }
89:    }
```


```diff
     /// @notice Deposits an asset lying in this NDC into its strategy
@@ -81,9 +81,8 @@ contract NodeDelegator is INodeDelegator, LRTConfigRoleChecker, PausableUpgradea
         onlySupportedAsset(asset)
         onlyLRTManager
     {
-        address lrtDepositPool = lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL);

-        if (!IERC20(asset).transfer(lrtDepositPool, amount)) {
+        if (!IERC20(asset).transfer(lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL), amount)) {
             revert TokenTransferFailed();
         }
     }
```

### No need to cache `lrtConfig.assetStrategy(asset)`

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L121-L124

```solidity
File: /src/NodeDelegator.sol
121:    function getAssetBalance(address asset) external view override returns (uint256) {
122:        address strategy = lrtConfig.assetStrategy(asset);
123:        return IStrategy(strategy).userUnderlyingView(address(this));
124:    }
```

```diff
     function getAssetBalance(address asset) external view override returns (uin
t256) {
-        address strategy = lrtConfig.assetStrategy(asset);
-        return IStrategy(strategy).userUnderlyingView(address(this));
+        return IStrategy(lrtConfig.assetStrategy(asset)).userUnderlyingView(address(this));
     }
```

### No need to cache `nodeDelegatorQueue[ndcIndex]`

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L183-L197

```solidity
File: /src/LRTDepositPool.sol
183:    function transferAssetToNodeDelegator(

192:    {
193:        address nodeDelegator = nodeDelegatorQueue[ndcIndex];
194:        if (!IERC20(asset).transfer(nodeDelegator, amount)) {
195:            revert TokenTransferFailed();
196:        }
197:    }
```

```diff
@@ -190,8 +190,7 @@ contract LRTDepositPool is ILRTDepositPool, LRTConfigRoleChecker, PausableUpgrad
         onlyLRTManager
         onlySupportedAsset(asset)
     {
-        address nodeDelegator = nodeDelegatorQueue[ndcIndex];
-        if (!IERC20(asset).transfer(nodeDelegator, amount)) {
+        if (!IERC20(asset).transfer(nodeDelegatorQueue[ndcIndex], amount)) {
             revert TokenTransferFailed();
         }
     }
```


## Caching the length of calldata increases gas cost instead of reducing it

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L162-L177
```solidity
File: /src/LRTDepositPool.sol
162:    function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {
163:        uint256 length = nodeDelegatorContracts.length;
164:        if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
165:            revert MaximumNodeDelegatorCountReached();
166:        }

168:        for (uint256 i; i < length;) {
169:            UtilLib.checkNonZeroAddress(nodeDelegatorContracts[i]);
170:            nodeDelegatorQueue.push(nodeDelegatorContracts[i]);
171:            emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]);
172:            unchecked {
173:                ++i;
174:            }
175:        }
176:    }
```

```diff
     function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {
         uint256 length = nodeDelegatorContracts.length;
-        if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
+        if (nodeDelegatorQueue.length + nodeDelegatorContracts.length > maxNodeDelegatorCount) {
             revert MaximumNodeDelegatorCountReached();
         }

-        for (uint256 i; i < length;) {
+        for (uint256 i; i < nodeDelegatorContracts.length;) {
```