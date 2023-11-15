## **G-1** | Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration. 

```diff
diff --git a/LRTDepositPool.sol b/aLRTDepositPool.sol
index 29ea4d2..84a3c0b 100644
--- a/LRTDepositPool.sol
+++ b/aLRTDepositPool.sol
@@ -160,19 +161,21 @@ contract LRTDepositPool is ILRTDepositPool, LRTConfigRoleChecker, PausableUpgrad
     /// @dev only callable by LRT manager
     /// @param nodeDelegatorContracts Array of NodeDelegator contract addresses
     function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {
-        uint256 length = nodeDelegatorContracts.length;
-        if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
+        //uint256 length = nodeDelegatorContracts.length;
+        //@audit use do while
+        if (nodeDelegatorQueue.length + nodeDelegatorContracts.length > maxNodeDelegatorCount) {
             revert MaximumNodeDelegatorCountReached();
         }
-
-        for (uint256 i; i < length;) {
+        uint256 i;
+        do{
+        //for (uint256 i; i < nodeDelegatorContracts.length;) {
             UtilLib.checkNonZeroAddress(nodeDelegatorContracts[i]);
             nodeDelegatorQueue.push(nodeDelegatorContracts[i]);
             emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]);
             unchecked {
                 ++i;
             }
-        }
+        }while(i<nodeDelegatorContracts.length);
     }

```

> Gas diff

```diff
 | src/LRTDepositPool.sol:LRTDepositPool contract |                 |       |        |        |         |
 |------------------------------------------------|-----------------|-------|--------|--------|---------|
 | Deployment Cost                                | Deployment Size |       |        |        |         |
-| 1686453                                        | 8550            |       |        |        |         |
+| 1684453                                        | 8540            |       |        |        |         |
 | Function Name                                  | min             | avg   | median | max    | # calls |
-| addNodeDelegatorContractToQueue                | 2608            | 53488 | 26786  | 112022 | 18      |
+| addNodeDelegatorContractToQueue                | 2599            | 53432 | 26734  | 111942 | 18      |

```

```diff
diff --git a/LRTOracle.sol b/aLRTOracle.sol
index 48c981a..8fef1e3 100644
--- a/LRTOracle.sol
+++ b/aLRTOracle.sol
@@ -62,8 +62,9 @@ contract LRTOracle is ILRTOracle, LRTConfigRoleChecker, PausableUpgradeable {
 
         address[] memory supportedAssets = lrtConfig.getSupportedAssetList();
         uint256 supportedAssetCount = supportedAssets.length;
-
-        for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
+        //@audit use do while
+        uint16 asset_idx;
+        do{//for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
             address asset = supportedAssets[asset_idx];
             uint256 assetER = getAssetPrice(asset);
 
@@ -73,7 +74,7 @@ contract LRTOracle is ILRTOracle, LRTConfigRoleChecker, PausableUpgradeable {
             unchecked {
                 ++asset_idx;
             }
-        }
+        }while(asset_idx < supportedAssetCount);
 
         return totalETHInPool / rsEthSupply;
     }

```

```diff
 | src/LRTOracle.sol:LRTOracle contract |                 |       |        |       |         |
 |--------------------------------------|-----------------|-------|--------|-------|---------|
 | Deployment Cost                      | Deployment Size |       |        |       |         |
-| 1089217                              | 5566            |       |        |       |         |
+| 1088017                              | 5560            |       |        |       |         |
 | Function Name                        | min             | avg   | median | max   | # calls |
 | assetPriceOracle                     | 558             | 1558  | 1558   | 2558  | 2       |
 | getAssetPrice                        | 15000           | 17670 | 17670  | 20341 | 2       |
-| getRSETHPrice                        | 15460           | 36558 | 36558  | 57656 | 2       |
+| getRSETHPrice                        | 15460           | 36520 | 36520  | 57581 | 2       |

```


```diff
diff --git a/NodeDelegator.sol b/aNodeDelegator.sol
index bba20ca..e2055f1 100644
--- a/NodeDelegator.sol
+++ b/aNodeDelegator.sol
@@ -105,14 +105,16 @@ contract NodeDelegator is INodeDelegator, LRTConfigRoleChecker, PausableUpgradea
         uint256 strategiesLength = strategies.length;
         assets = new address[](strategiesLength);
         assetBalances = new uint256[](strategiesLength);
-
-        for (uint256 i = 0; i < strategiesLength;) {
+        //@audit Use Do while
+        uint256 i;
+        do{
+        //for (uint256 i = 0; i < strategiesLength;) {
             assets[i] = address(IStrategy(strategies[i]).underlyingToken());
             assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
             unchecked {
                 ++i;
             }
-        }
+        }while(i < strategiesLength);
     }
 

```


> Gas diff

```diff
 | src/NodeDelegator.sol:NodeDelegator contract |                 |       |        |        |         |
 |----------------------------------------------|-----------------|-------|--------|--------|---------|
 | Deployment Cost                              | Deployment Size |       |        |        |         |
-| 1668632                                      | 8461            |       |        |        |         |
+| 1667432                                      | 8454            |       |        |        |         |
 | Function Name                                | min             | avg   | median | max    | # calls |
 | depositAssetIntoStrategy                     | 569             | 77147 | 90253  | 133653 | 7       |
 | getAssetBalance                              | 7387            | 7387  | 7387   | 7387   | 1       |
-| getAssetBalances                             | 24953           | 24953 | 24953  | 24953  | 1       |
+| getAssetBalances                             | 24898           | 24898 | 24898  | 24898  | 1       |

```

## **G-2** | Use assembly to write address storage values 


By using assembly, we can directly interact with the storage slot of the storedAddress variable, allowing us to efficiently write and read address values in storage.

```diff
diff --git a/LRTConfig.sol b/aLRTConfig.sol
index 41b09e9..9ac9df5 100644
--- a/LRTConfig.sol
+++ b/aLRTConfig.sol
@@ -63,7 +63,6 @@ contract LRTConfig is ILRTConfig, AccessControlUpgradeable {
         _addNewSupportedAsset(cbETH, 100_000 ether);
 
         _grantRole(DEFAULT_ADMIN_ROLE, admin);

         rsETH = rsETH_;
     }
 
@@ -141,9 +140,13 @@ contract LRTConfig is ILRTConfig, AccessControlUpgradeable {
     //////////////////////////////////////////////////////////////*/
     /// @dev Sets the rsETH contract address. Only callable by the admin
     /// @param rsETH_ rsETH contract address
+    //@audit Use assembly to write address storage values 
     function setRSETH(address rsETH_) external onlyRole(DEFAULT_ADMIN_ROLE) {
         UtilLib.checkNonZeroAddress(rsETH_);
-        rsETH = rsETH_;
+        assembly{
+            sstore(rsETH.slot,rsETH_)
+        }
+        //rsETH = rsETH_;
     }

```

> Gas diff 

```diff
 | src/LRTConfig.sol:LRTConfig contract |                 |        |        |        |         |
 |--------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                      | Deployment Size |        |        |        |         |
-| 1316026                              | 6686            |        |        |        |         |
+| 1302813                              | 6620            |        |        |        |         |
 | Function Name                        | min             | avg    | median | max    | # calls |
 | DEFAULT_ADMIN_ROLE                   | 260             | 260    | 260    | 260    | 1       |
 | addNewSupportedAsset                 | 2804            | 28343  | 17355  | 75860  | 4       |
@@ -79,7 +79,7 @@
 | isSupportedAsset                     | 561             | 1530   | 561    | 2561   | 66      |
 | rsETH                                | 348             | 2125   | 2348   | 2348   | 9       |
 | setContract                          | 1001            | 24167  | 24535  | 29678  | 93      |
-| setRSETH                             | 1002            | 2382   | 1002   | 29694  | 27      |
+| setRSETH                             | 863             | 2257   | 863    | 29694  | 27      |
```