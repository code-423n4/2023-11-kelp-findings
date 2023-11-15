# KELP PROTOCOL GAS OPTIMISATIONS



## INTRODUCTION
Some of optimizations underwent benchmarking using the protocol's tests, specifically employing the following configuration: `solc version 0.8.21`, `optimizer enabled,` and `10_000` runs. For optimizations that were not subjected to benchmarking, their rationale is elucidated through EVM gas expenses and opcodes.

Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime. 

Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate some issue explanations."





## [G-01] Multiple mappings can be replaced with a single struct mapping

### 1 Instance
1. #### The mappings `mapping(address token => bool isSupported) public isSupportedAsset`, `mapping(address token => uint256 amount) public depositLimitByAsset` and `mapping(address token => address strategy) public override assetStrategy` should be converted to a single address to struct mapping
- https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L15-#L17


The mappings `mapping(address token => bool isSupported) public isSupportedAsset`, `mapping(address token => uint256 amount) public depositLimitByAsset` and `mapping(address token => address strategy) public override assetStrategy` should be converted to a single address to struct mapping. Implementing this would reduce the number of storage slots for these mappings (i.e the storage slots used in the keccak to determine the location of the mapping value) from 3 to 1 and the slots required for the values from 3 to 2 (if the newly defined struct is packed) therefore we would save 20k gas in avoiding `SSTORE` in setting each of these storage slots. In functions such as the `_addNewSupportedAsset()` we would not have to calculate keep calculating the hash to read the values or write to `isSupportedAsset[asset]` and `depositLimitByAsset[asset]` as we can cache the struct into a local `storage` variable, read and modify without having to re-calculate mappings hash. The diff below shows how the code could be refcatored;

```solidity
file: src/LRTConfig.sol

15:    mapping(address token => bool isSupported) public isSupportedAsset;
16:    mapping(address token => uint256 amount) public depositLimitByAsset;
17:    mapping(address token => address strategy) public override assetStrategy;
```

```diff
diff --git a/src/LRTConfig.sol b/src/LRTConfig.sol
index 41b09e9..355c447 100644
--- a/src/LRTConfig.sol
+++ b/src/LRTConfig.sol
@@ -10,11 +10,15 @@ import { AccessControlUpgradeable } from "@openzeppelin/contracts-upgradeable/ac
 /// @title LRTConfig - LRT Config Contract
 /// @notice Handles LRT configuration
 contract LRTConfig is ILRTConfig, AccessControlUpgradeable {
+
+    struct Token {
+        bool isSupported;
+        uint256 amount;
+        address strategy;
+    }
     mapping(bytes32 tokenKey => address tokenAddress) public tokenMap;
     mapping(bytes32 contractKey => address contractAddress) public contractMap;
-    mapping(address token => bool isSupported) public isSupportedAsset;
-    mapping(address token => uint256 amount) public depositLimitByAsset;
-    mapping(address token => address strategy) public override assetStrategy;
+    mapping(address token => Token token) public tokens;
```
```
Estimated gas saved: 60k gas units
```


## [G-02] Multiple accesses of an array should use a local variable cache
The instances below point to the second+ access of a value inside an array, within a function. Caching an array's value avoids re-calculating the array offsets into memory.
#### Please note these Instances were not included in the bots reports.

### 2 Instances
1. #### Cache `nodeDelegatorContracts[i]`
- https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L169-#L171

We can reduce the gas usage of the `addNodeDelegatorContractToQueue()` function by caching the value of  `nodeDelegatorContracts[i]` array in a local variable during the loop iterations and using that local variable for subsequent `nodeDelegatorContracts[i]` reads. Implementing this would help avoid the EVM having to re-calculate the array's offsets into memory rather it does a much cheaper stack read. The diff below shows how the code should be refactored:

```solidity
file: src/LRTDepositPool.sol

162:    function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {
163:        uint256 length = nodeDelegatorContracts.length;
164:        if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
165:            revert MaximumNodeDelegatorCountReached();
166:        }
167:
168:        for (uint256 i; i < length;) {
169:            UtilLib.checkNonZeroAddress(nodeDelegatorContracts[i]); //@audit cache nodeDelegatorContracts[i]
170:            nodeDelegatorQueue.push(nodeDelegatorContracts[i]);
171:            emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]);
172:            unchecked {
173:                ++i;
174:            }
175:        }
176:    }
```

```diff
diff --git a/src/LRTDepositPool.sol b/src/LRTDepositPool.sol
index 29ea4d2..43969ac 100644
--- a/src/LRTDepositPool.sol
+++ b/src/LRTDepositPool.sol
@@ -166,9 +166,10 @@ contract LRTDepositPool is ILRTDepositPool, LRTConfigRoleChecker, PausableUpgrad
         }

         for (uint256 i; i < length;) {
-            UtilLib.checkNonZeroAddress(nodeDelegatorContracts[i]);
-            nodeDelegatorQueue.push(nodeDelegatorContracts[i]);
-            emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]);
+            address nodeDelegatorContract = nodeDelegatorContracts[i];
+            UtilLib.checkNonZeroAddress(nodeDelegatorContract);
+            nodeDelegatorQueue.push(nodeDelegatorContract);
+            emit NodeDelegatorAddedinQueue(nodeDelegatorContract);
             unchecked {
                 ++i;
             }
```
#### Gas saving for `addNodeDelegatorContractToQueue()` function obtained via protocol test: 1326 gas units
BEFORE
```
test/LRTDepositPoolTest.t.sol:LRTDepositPoolAddNodeDelegatorContractToQueue
[PASS] test_AddNodeDelegatorContractToQueue() (gas: 145254)
```
AFTER
```
test/LRTDepositPoolTest.t.sol:LRTDepositPoolAddNodeDelegatorContractToQueue
[PASS] test_AddNodeDelegatorContractToQueue() (gas: 143928)
```


2. #### Cache `IStrategy(strategies[i])`
- https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L110-#L111

We can reduce the gas usage of the `getAssetBalances()` function by caching the value of  `IStrategy(strategies[i])` in a local variable during the loop iterations and using that local variable for subsequent `IStrategy(strategies[i])` reads. Implementing this would help avoid the EVM having to re-calculate the array's offsets into memory and having to perform the type conversion twice rather it does a much cheaper stack read. The diff below shows how the code should be refactored:

```solidity
file: src/NodeDelegator.sol

94:     function getAssetBalances()
95:         external
96:         view
97:         override
98:         returns (address[] memory assets, uint256[] memory assetBalances)
99:     {
100:        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);
101:
102:        (IStrategy[] memory strategies,) =
103:            IEigenStrategyManager(eigenlayerStrategyManagerAddress).getDeposits(address(this));
104:
105:        uint256 strategiesLength = strategies.length;
106:        assets = new address[](strategiesLength);
107:        assetBalances = new uint256[](strategiesLength);
108:
109:        for (uint256 i = 0; i < strategiesLength;) {
110:            assets[i] = address(IStrategy(strategies[i]).underlyingToken());  //@audit cache IStrategy(strategies[i])
111:            assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
112:            unchecked {
113:                ++i;
114:            }
115:        }
116:    }
```

```diff
diff --git a/src/NodeDelegator.sol b/src/NodeDelegator.sol
index bba20ca..6451d1c 100644
--- a/src/NodeDelegator.sol
+++ b/src/NodeDelegator.sol
@@ -107,8 +107,9 @@ contract NodeDelegator is INodeDelegator, LRTConfigRoleChecker, PausableUpgradea
         assetBalances = new uint256[](strategiesLength);

         for (uint256 i = 0; i < strategiesLength;) {
-            assets[i] = address(IStrategy(strategies[i]).underlyingToken());
-            assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
+            IStrategy strategy = IStrategy(strategies[i]);
+            assets[i] = address(strategy.underlyingToken());
+            assetBalances[i] = strategy.userUnderlyingView(address(this));
             unchecked {
                 ++i;
             }
```
#### Gas saving for `getAssetBalances()` function obtained via protocol test: 118 gas units
BEFORE
```
test/NodeDelegatorTest.t.sol:NodeDelegatorGetAssetBalances
[PASS] test_GetAssetBalances() (gas: 277672)
```
AFTER
```
test/NodeDelegatorTest.t.sol:NodeDelegatorGetAssetBalances
[PASS] test_GetAssetBalances() (gas: 277554)
```




## [G-03] Initializers can be marked as payable to save deployment gas
Payable functions cost less gas to execute, because the compiler does not have to add extra checks to ensure that no payment is provided. Initializers can be safely marked as payable, because only the deployer or the factory contract would call the function without carrying any funds. We can reduce the function cost by `15` gas units by implementing this change.

### 6 Instances
1. #### Make `LRTConfig.initialize()` function a `payable` function
- https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L41-#L68

```solidity
file: src/LRTConfig.sol

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
.
.
.
68:    }
```

```diff
diff --git a/src/LRTConfig.sol b/src/LRTConfig.sol
index 41b09e9..cea7a3e 100644
--- a/src/LRTConfig.sol
+++ b/src/LRTConfig.sol
@@ -46,6 +46,7 @@ contract LRTConfig is ILRTConfig, AccessControlUpgradeable {
     function initialize(
         address admin,
         address stETH,
         address rETH,
         address cbETH,
         address rsETH_
     )
         external
+        payable
         initializer
     {
         UtilLib.checkNonZeroAddress(admin);
```
```
Estimated gas saved: 15 gas units
```

2. #### Make `LRTDepositPool.initialize()` function a `payable` function
- https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L31-#L38

```solidity
file: src/LRTDepositPool.sol

31:    function initialize(address lrtConfigAddr) external initializer {
32:        UtilLib.checkNonZeroAddress(lrtConfigAddr);
33:        __Pausable_init();
34:        __ReentrancyGuard_init();
35:        maxNodeDelegatorCount = 10;
36:        lrtConfig = ILRTConfig(lrtConfigAddr);
37:        emit UpdatedLRTConfig(lrtConfigAddr);
38:    }
```

```diff
diff --git a/src/LRTDepositPool.sol b/src/LRTDepositPool.sol
index 29ea4d2..f3b175d 100644
--- a/src/LRTDepositPool.sol
+++ b/src/LRTDepositPool.sol
@@ -28,7 +28,7 @@ contract LRTDepositPool is ILRTDepositPool, LRTConfigRoleChecker, PausableUpgrad

     /// @dev Initializes the contract
     /// @param lrtConfigAddr LRT config address
-    function initialize(address lrtConfigAddr) external initializer {
+    function initialize(address lrtConfigAddr) external payable initializer {
         UtilLib.checkNonZeroAddress(lrtConfigAddr);
         __Pausable_init();
         __ReentrancyGuard_init();

```
```
Estimated gas saved: 15 gas units
```

3. #### Make `NodeDelegator.initialize()` function a `payable` function
- https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L26-#L33

```solidity
file: src/NodeDelegator.sol

26:    function initialize(address lrtConfigAddr) external initializer {
27:        UtilLib.checkNonZeroAddress(lrtConfigAddr);
28:        __Pausable_init();
29:        __ReentrancyGuard_init();
30:
31:        lrtConfig = ILRTConfig(lrtConfigAddr);
32:        emit UpdatedLRTConfig(lrtConfigAddr);
33:    }
```

```diff
diff --git a/src/NodeDelegator.sol b/src/NodeDelegator.sol
index bba20ca..79a404b 100644
--- a/src/NodeDelegator.sol
+++ b/src/NodeDelegator.sol
@@ -23,7 +23,7 @@ contract NodeDelegator is INodeDelegator, LRTConfigRoleChecker, PausableUpgradea

     /// @dev Initializes the contract
     /// @param lrtConfigAddr LRT config address
-    function initialize(address lrtConfigAddr) external initializer {
+    function initialize(address lrtConfigAddr) external payable initializer {
         UtilLib.checkNonZeroAddress(lrtConfigAddr);
         __Pausable_init();
         __ReentrancyGuard_init();
```
```
Estimated gas saved: 15 gas units
```

4. #### Make `LRTOracle.initialize()` function a `payable` function
- https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L29-#L35

```solidity
file: src/LRTOracle.sol

29:    function initialize(address lrtConfigAddr) external initializer {
30:        UtilLib.checkNonZeroAddress(lrtConfigAddr);
31:        __Pausable_init();
32:
33:        lrtConfig = ILRTConfig(lrtConfigAddr);
34:        emit UpdatedLRTConfig(lrtConfigAddr);
35:    }
```

```diff
diff --git a/src/LRTOracle.sol b/src/LRTOracle.sol
index 48c981a..72f8638 100644
--- a/src/LRTOracle.sol
+++ b/src/LRTOracle.sol
@@ -26,7 +26,7 @@ contract LRTOracle is ILRTOracle, LRTConfigRoleChecker, PausableUpgradeable {

     /// @dev Initializes the contract
     /// @param lrtConfigAddr LRT config address
-    function initialize(address lrtConfigAddr) external initializer {
+    function initialize(address lrtConfigAddr) external payable initializer {
         UtilLib.checkNonZeroAddress(lrtConfigAddr);
         __Pausable_init();
```
```
Estimated gas saved: 15 gas units
```


5. #### Make `RSETH.initialize()` function a `payable` function
- https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L32-#L42

```solidity
file: src/RSETH.sol

32:    function initialize(address admin, address lrtConfigAddr) external initializer {
33:        UtilLib.checkNonZeroAddress(admin);
34:        UtilLib.checkNonZeroAddress(lrtConfigAddr);
35:
36:        __ERC20_init("rsETH", "rsETH");
37:        __Pausable_init();
38:        __AccessControl_init();
39:        _grantRole(DEFAULT_ADMIN_ROLE, admin);
40:        lrtConfig = ILRTConfig(lrtConfigAddr);
41:        emit UpdatedLRTConfig(lrtConfigAddr);
42:    }
```

```diff
diff --git a/src/RSETH.sol b/src/RSETH.sol
index d063cf5..698f18f 100644
--- a/src/RSETH.sol
+++ b/src/RSETH.sol
@@ -29,7 +29,7 @@ contract RSETH is
     /// @dev Initializes the contract
     /// @param admin Admin address
     /// @param lrtConfigAddr LRT config address
-    function initialize(address admin, address lrtConfigAddr) external initializer {
+    function initialize(address admin, address lrtConfigAddr) external payable initializer {
         UtilLib.checkNonZeroAddress(admin);
         UtilLib.checkNonZeroAddress(lrtConfigAddr);
```
```
Estimated gas saved: 15 gas units
```


6. #### Make `ChainlinkPriceOracle.initialize()` function a `payable` function
- https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L27-#L32
```solidity
file: src/oracles/ChainlinkPriceOracle.sol

27:    function initialize(address lrtConfig_) external initializer {
28:        UtilLib.checkNonZeroAddress(lrtConfig_);
29:
30:        lrtConfig = ILRTConfig(lrtConfig_);
31:        emit UpdatedLRTConfig(lrtConfig_);
32:    }
```

```diff
diff --git a/src/oracles/ChainlinkPriceOracle.sol b/src/oracles/ChainlinkPriceOracle.sol
index 41798a2..8ab57e8 100644
--- a/src/oracles/ChainlinkPriceOracle.sol
+++ b/src/oracles/ChainlinkPriceOracle.sol
@@ -24,7 +24,7 @@ contract ChainlinkPriceOracle is IPriceFetcher, LRTConfigRoleChecker, Initializa

     /// @dev Initializes the contract
     /// @param lrtConfig_ LRT config address
-    function initialize(address lrtConfig_) external initializer {
+    function initialize(address lrtConfig_) external payable initializer {
         UtilLib.checkNonZeroAddress(lrtConfig_);

         lrtConfig = ILRTConfig(lrtConfig_);
```
```
Estimated gas saved: 15 gas units
```




## [G-04] Declaring Unnecessary variables
some varibles were defined even though they are used once. Not defining variables can reduce gas cost and contract size.

### 4 Instances
1. #### Declaration of `rsethAmountMinted` unnecessary
- https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L141

In the `depositAsset()` function below rather than having to declare the variable `rsethAmountMinted` to cache the return value of `_mintRsETH()` function we can use call the `_mintRsETH()` function directly in the emit `AssetDeposit()` statement. The diff below shows how the code could be refactored:

```solidity
file: src/LRTDepositPool.sol

119:   function depositAsset(
120:        address asset,
121:        uint256 depositAmount
122:    )
123:        external
124:        whenNotPaused
125:        nonReentrant
126:        onlySupportedAsset(asset)
127:    {
128:        // checks
129:        if (depositAmount == 0) {
130:            revert InvalidAmount();
131:        }
132:        if (depositAmount > getAssetCurrentLimit(asset)) {
133:            revert MaximumDepositLimitReached();
134:        }
135:
136:        if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) {
137:            revert TokenTransferFailed();
138:        }
139:
140:        // interactions
141:        uint256 rsethAmountMinted = _mintRsETH(asset, depositAmount);   //@audit declaration of `rsethAmountMinted` unnecessary
142:
143:        emit AssetDeposit(asset, depositAmount, rsethAmountMinted);
144:    }
```

```diff
diff --git a/src/LRTDepositPool.sol b/src/LRTDepositPool.sol
index 29ea4d2..d7cc2d4 100644
--- a/src/LRTDepositPool.sol
+++ b/src/LRTDepositPool.sol
@@ -137,10 +137,7 @@ contract LRTDepositPool is ILRTDepositPool, LRTConfigRoleChecker, PausableUpgrad
             revert TokenTransferFailed();
         }

-        // interactions
-        uint256 rsethAmountMinted = _mintRsETH(asset, depositAmount);
-
-        emit AssetDeposit(asset, depositAmount, rsethAmountMinted);
+        emit AssetDeposit(asset, depositAmount, _mintRsETH(asset, depositAmount));
     }
```
```
Estimated gas saved: 3 gas units
```

2. #### Declaration of `nodeDelegator` unnecessary
- https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L193

The variable `nodeDelegator` was declared and used as a state variable cache for `nodeDelegatorQueue[ndcIndex]` but `nodeDelegator` was only used once in the function in the `IERC20(asset).transfer(nodeDelegator, amount)` statement, we could use `nodeDelegatorQueue[ndcIndex]` directly instead thereby making the declaration of `nodeDelegator` unnecessary. The code should be refactored as shown in the diff below:

```solidity
file: src/LRTDepositPool.sol

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
194:        if (!IERC20(asset).transfer(nodeDelegator, amount)) {
195:            revert TokenTransferFailed();
196:        }
197:    }
```

```diff
diff --git a/src/LRTDepositPool.sol b/src/LRTDepositPool.sol
index 29ea4d2..2a6232c 100644
--- a/src/LRTDepositPool.sol
+++ b/src/LRTDepositPool.sol
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
```
Estimated gas saved: 3 gas units.
```






## CONCLUSIONS
As you embark on incorporating the recommended optimizations, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.

