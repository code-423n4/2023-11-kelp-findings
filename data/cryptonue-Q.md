
# [L] Lack of duplicate check on `nodeDelegatorQueue`

There is no unique check on nodeDelegatorQueue thus open for potential duplicates of data. Moreover there is no function to remove the nodeDelegatorQueue.

```js
File: LRTDepositPool.sol
162:     function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {
163:         uint256 length = nodeDelegatorContracts.length;
164:         if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
165:             revert MaximumNodeDelegatorCountReached();
166:         }
167:
168:         for (uint256 i; i < length;) {
169:             UtilLib.checkNonZeroAddress(nodeDelegatorContracts[i]);
170:             nodeDelegatorQueue.push(nodeDelegatorContracts[i]);
171:             emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]);
172:             unchecked {
173:                 ++i;
174:             }
175:         }
176:     }
```

# [NC] redundant variable

as seen there is this code `IERC20 token = IERC20(asset);` which doesn't do anything much, redundant variable assignment, we can just use the `asset` directly rather than create a `token` variable.

```js
File: NodeDelegator.sol
51:     function depositAssetIntoStrategy(address asset)
52:         external
53:         override
54:         whenNotPaused
55:         nonReentrant
56:         onlySupportedAsset(asset)
57:         onlyLRTManager
58:     {
59:         address strategy = lrtConfig.assetStrategy(asset);
60:         IERC20 token = IERC20(asset); // @audit why is this?
61:         address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);
62:
63:         uint256 balance = token.balanceOf(address(this));
64:
65:         emit AssetDepositIntoStrategy(asset, strategy, balance);
66:
67:         IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
68:     }
```