# L-01 : `shares ` are not returned ! 

While depositing  into strategy , `eigenlayerStrategyManagerAddress` contracts returns the amount of shares minted for that particular deposit . However catching that return value got skipped here in this line : 
```solidity 
  IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L67
## recommendation : 
I recommend to catch the returned amount and return to the user for better transparency . 
```diff
-      function depositAssetIntoStrategy(address asset)
-        //...
-    {

+      function depositAssetIntoStrategy(address asset)
+        //...
+       returns (uint256 shares ) 
+    {

   //....

-     IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
+     return   IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);


```


# L-02 : 