#  L1. Dynamic array used in `LRTConfig.sol`
## Title
Unbounded loops used in `LRTConfig.sol`
## Impact
Description: The `getSupportedAssetList` function in the provided smart contract returns a dynamic array containing the list of supported assets. This function currently does not limit the number of assets it can return. In scenarios where the number of supported assets grows large, calling this function could require more gas than is available in a block. This situation would make the function fail and could potentially make parts of the contract unusable or cause issues for interfaces and other contracts that depend on this data.

Line Number: Not directly present but implied in functions that may iterate over `getSupportedAssetList`.
Recommended Fix: Consider implementing a pattern to limit the size of this list or handle large lists in a gas-efficient manner. https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L135
```
function getSupportedAssetList() external view override returns (address[] memory) {
        return supportedAssetList;
    }
```

## Proof of Concept
With amount array length of 15 and above, it is highly likely to hit the gasLimit.
## Tools Used
Manual Review

## Recommended Mitigation Steps
Pagination: Modify the getSupportedAssetList function to return a subset of the supported assets based on pagination parameters (e.g., start index and count). This approach allows callers to retrieve all assets in smaller, manageable chunks.

Example Implementation:

```solidity
function getSupportedAssetList(uint start, uint count) external view returns (address[] memory) {
    uint totalAssets = supportedAssetList.length;
    if (start >= totalAssets) {
        return new address[](0);
    }

    uint end = start + count;
    if (end > totalAssets) {
        end = totalAssets;
    }

    address[] memory assets = new address[](end - start);
    for (uint i = start; i < end; i++) {
        assets[i - start] = supportedAssetList[i];
    }
    return assets;
}



```

# L2. Reentrancy Guard  `transferAssetToNodeDelegator`
## Title
Reentrancy Guard  `transferAssetToNodeDelegator`

## Impact
The `depositAsset` function is correctly using the `nonReentrant` modifier to prevent reentrancy attacks. However, other external functions like `transferAssetToNodeDelegator` that interact with external contracts (IERC20.transfer) should also use `nonReentrant` to prevent potential reentrancy vulnerabilities.
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L183

```solidity
function transferAssetToNodeDelegator(
    uint256 ndcIndex,
    address asset,
    uint256 amount
)
    external
    nonReentrant
    onlyLRTManager
    onlySupportedAsset(asset)
{
    address nodeDelegator = nodeDelegatorQueue[ndcIndex];
    if (!IERC20(asset).transfer(nodeDelegator, amount)) {
        revert TokenTransferFailed();
    }
}

```
## Proof of Concept

## Tools Used
Manual Review

## Recommended Mitigation Steps
Apply the `nonReentrant` modifier to all external/public functions that make external calls or token transfers.
```solidity
function transferAssetToNodeDelegator(
    uint256 ndcIndex,
    address asset,
    uint256 amount
)
    external
    nonReentrant
    onlyLRTManager
    onlySupportedAsset(asset)
{
    address nodeDelegator = nodeDelegatorQueue[ndcIndex];
    if (!IERC20(asset).transfer(nodeDelegator, amount)) {
        revert TokenTransferFailed();
    }
}
```


