## L-01 No check on whether existing strategy has funds before changing them 

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L109-L122

Protocol does not check whether a strategy has funds in them before changing it. This could present some issues when withdrawing funds.



## L-02 No check on whether a strategy has been whitelisted in Eigen Layer 
https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L109-L122
In Eigen Layer's deposit function, there is a check to make sure that a certain strategy is whitelisted. Kelp does not make that same check when it calls Eigen's deposit function. Therefore, there can be situations where a strategy is approved on Kelp but removed from Eigen Layer's whitelist, causing unexpected reverts 



## L-03 Library is not needed

The Library in this function does not add much utility especially when it come to asset management as the constants are fixed and the admin can add new assets without using the library


## L-04 LRTOracle contracts are not pauseable 

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L31


None of the oracle functions in LRTOracle.sol are pauseable even though the contract inherits pauseable_init. 

## L-05 Admin can renounce ownership 

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/3d4c0d5741b131c231e558d7a6213392ab3672a5/contracts/access/AccessControlUpgradeable.sol#L186-L190

Admin should not be able to renounce ownership as it would break several core contract functions


## L-06 Malicious Actor can make a donation attack to cause deposits to fail

```
   if (depositAmount > getAssetCurrentLimit(asset)) {
            revert MaximumDepositLimitReached();
        }
```

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L132-L133

 The deposit function will fail if it exceeds the assetdeposit limit. It uses the following calculation for the limit 


```
lrtConfig.depositLimitByAsset(asset) - getTotalAssetDeposits(asset)
```

Which is derived from the function getAssetDistributionData shown below:


```
        // Question: is here the right place to have this? Could it be in LRTConfig?
        assetLyingInDepositPool = IERC20(asset).balanceOf(address(this));


        uint256 ndcsCount = nodeDelegatorQueue.length;
        for (uint256 i; i < ndcsCount;) {
            assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
            unchecked {
                ++i;
            }
        }
```

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L79

The problem is that when calculating the limit it uses the balance of the contract as well as the node delegators.
That means that a bad actor can donate just enough of the asset to cause deposits to fail