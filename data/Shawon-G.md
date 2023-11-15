# Don't store data in a variable if it is used only once.

``` solidity 
file: utils/LRTDeposit.sol

81 uint256 ndcsCount = nodeDelegatorQueue.length;
        for (uint256 i; i < ndcsCount;) {
            assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
            unchecked {
                ++i;
            }
        }
    }
```
The value of `nodeDelegatorQueue.length` is used only once and caching it inside a uint256 variable would only use memory storage hence,
increasing the gas price.

Insdead, do this:

``` solidity
for (uint256 i; i < nodeDelegatorQueue.length;) {
            assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
            unchecked {
                ++i;
            }
        }
    }
```