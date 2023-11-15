# 1. maxNodeDelegatorCount does not get enforced

### Impact

There are a few problems with the usage of `maxNodeDelegatorCount` in `LRTDepositPool.sol`.

First problem, consider the following:
- `maxNodeDelegatorCount = 10`
- 10 `NodeDelegators` have been added using `LRTDepositPool.addNodeDelegatorContractToQueue`
- `maxNodeDelegatorCount` gets updated to `8`
- However, `LRTDepositPool.getAssetDistributionData` will still use all 10 `NodeDelegators`, despite the max being set to `8`:

[LRTDepositPool.sol#L81-L86](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L81-L86)
```javascript
 uint256 ndcsCount = nodeDelegatorQueue.length;
        for (uint256 i; i < ndcsCount;) {
            assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
            unchecked {
                ++i;
```


Second problem, consider the following:
First problem, consider the following:
- `maxNodeDelegatorCount = 10`
- 10 `NodeDelegators` have been added using `LRTDepositPool.addNodeDelegatorContractToQueue`
- `maxNodeDelegatorCount` gets updated to `8`
- However, it is still possible to send funds from `LRTDepositPool` to any `NodeDelegator` that has ever been added using `LRTDepositPool.transferAssetToNodeDelegator`

[LRTDepositPool.sol#L193-195](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L193-195)
```javascript
address nodeDelegator = nodeDelegatorQueue[ndcIndex];
        if (!IERC20(asset).transfer(nodeDelegator, amount)) {
            revert TokenTransferFailed();
```

These two scenarios should not be possible.

### Tools used
Manual Review

### Recommended Mitigation Steps

Either remove `NodeDelegators` from the `nodeDelegatorQueue` when setting a `maxNodeDelegatorCount < currentmaxNodeDelegatorCount`, or, remove the functionality of being able to set a `maxNodeDelegatorCount` that is less than the current `maxNodeDelegatorCount`.

# 2. Incorrect access control in 'addNodeDelegatorToQueue'

### Impact
```javascript
    /// @notice add new node delegator contract addresses
    /// @dev only callable by LRT manager
    /// @param nodeDelegatorContracts Array of NodeDelegator contract addresses
    // @audit should be only callable by LRT Manager but is onlyLRTAdmin
    function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {
        uint256 length = nodeDelegatorContracts.length;
        if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
            revert MaximumNodeDelegatorCountReached();
        }

        for (uint256 i; i < length;) {
            UtilLib.checkNonZeroAddress(nodeDelegatorContracts[i]);
            nodeDelegatorQueue.push(nodeDelegatorContracts[i]);
            emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]);
            unchecked {
                ++i;
            }
        }
    }
```

As you can see from the comments of the devs comment it should be the LRT manager that should be able to call this, but currently its the LRTAdmin that can call it only because of the `onlyLRTAdmin` modifier.

### Recommended Mitigation Steps
Use `onlyLRTManager` instead of `onlyLRTAdmin`

# 3. A person can always mint rsETH for cheaper by minting just before stETH rebases

### Impact
`stETH` from Lido is a rebase token. [The Lido docs state](https://help.lido.fi/en/articles/5230610-what-is-steth):

```
The mechanism which updates the stETH balances every day is called a “rebase”. Every day at 12PM UTC the amount of stETH in your address will increase with the current APR.
```

This means that any user can just wait until `stETH` almost rebases and mint `rsETH` to receive more `rsETH` than he would have after the rebase