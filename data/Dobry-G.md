# GAS [1] Use named variables in function returns
###### 1 instance

##### When you don’t use the named variables in your return statement, you waste deployment gas.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol


47: function getTotalAssetDeposits(address asset) public view override returns (uint256 totalAssetDeposit) {
        (uint256 assetLyingInDepositPool, uint256 assetLyingInNDCs, uint256 assetStakedInEigenLayer) =
            getAssetDistributionData(asset);
        return (assetLyingInDepositPool + assetLyingInNDCs + assetStakedInEigenLayer);
    }

# [GAS-2] Operator >=/<= costs less gas than operator >/<
###### 5 instances
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol

132: if (depositAmount > getAssetCurrentLimit(asset)) {
164: if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
82: for (uint256 i; i < ndcsCount;) {
168: for (uint256 i; i < length;) {


https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol

98: returns (address[] memory assets, uint256[] memory assetBalances)