## lo-01 LRTConfig doesn't remove supported assets
The contract only allows the manager to add new supported assets by calling addNewSupportedAsset, but there is not function to remove an asset.

## lo-02 No upper limits on the amount of supported assets can lead to DOS
[Reference](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L80-L89)
### Impact 
The lack of upper bounds on the amount of supported assets can lead to getRSETHPrice reversion and lock the contract's minting functionalities.
### Proof of concept
There's no upper limit on the amount of supported assets that can be added at the LRTConfig contract. 
```solidity
    function _addNewSupportedAsset(address asset, uint256 depositLimit) private {
        UtilLib.checkNonZeroAddress(asset);
        if (isSupportedAsset[asset]) {
            revert AssetAlreadySupported();
        }
        isSupportedAsset[asset] = true;
        supportedAssetList.push(asset);
        depositLimitByAsset[asset] = depositLimit;
        emit AddedNewSupportedAsset(asset, depositLimit);

    }
```

If too many assets are supported, getRSETHPrice at the LRTOracle will revert due to the following code snippet, that represents a loop which iterations amount is determined by the total supported assets:
```solidity
       uint256 supportedAssetCount = supportedAssets.length;
        for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
            address asset = supportedAssets[asset_idx];
            uint256 assetER = getAssetPrice(asset);
            uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
            totalETHInPool += totalAssetAmt * assetER;
            unchecked {
                ++asset_idx;
            }
        }
```
### Mitigation
Introduce a check on the total amount of supported assets in order to protect the function from running loops iterate too many times.

## lo-03 getAssetPrice at the ChainlinkPriceOracle contract assumes all Chainlink Data Feeds return the same amount of decimals
getAssetPrice calls a Chainlink data feed and then returns this value to the deposit pool at getRsETHAmountToMint. The issue is not all data feed return the same amount of decimals. USD pairs return 8 decimals. ETH pairs usually return 18 decimals, but KNC and FTM data feeds return 15 decimals and LINK data feed returns 16. 

Notice how the function does not take into account the amount of decimals provided by the data feed:
function getAssetPrice(address asset) external view onlySupportedAsset(asset) returns (uint256) {
        return AggregatorInterface(assetPriceFeed[asset]).latestAnswer();
    }

It is understood that the currently EigenLayer integrated contracts actually do have 18 decimals data feed. This should be kept in mind in case integration is applied to new assets.

## lo-04 Can add the same node Delegator twice at an LRTDepositPoolContract
[Reference](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L162-L176)
When the LRTAdmin calls addNodeDelegatorContractToQueue, there are no checks to ensure the nodeDelegator is already included at the queue. 
```solidity
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
This creates a couple of side effects:
1. getAssetDistributionData will account for a balance more than once:
```solidity
function getAssetDistributionData(address asset)
        public
        view
        override
        onlySupportedAsset(asset)
        returns (uint256 assetLyingInDepositPool, uint256 assetLyingInNDCs, uint256 assetStakedInEigenLayer)
    {
...

        for (uint256 i; i < ndcsCount;) {
            assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
...
        }
    }
```
2. getTotalAssetDeposits at the LRTDepositPool will yield inflated balances.
3. getAssetCurrentLimit will subtract a value higher than it should.
```solidity
function getAssetCurrentLimit(address asset) public view override returns (uint256) {
return lrtConfig.depositLimitByAsset(asset) - getTotalAssetDeposits(asset);
    }
```
4. depositAsset will consider lower MaximumDepositLimits wrongly:
```solidity
function depositAsset(
        address asset,
        uint256 depositAmount
    ){
    ...
	if (depositAmount > getAssetCurrentLimit(asset)) {
            revert MaximumDepositLimitReached();
     }
     ...
}
```

5. As getTotalAssetDeposits is utilized at LRTOracle, the oracle will end up providing wrong rsETH prices as well.

Since this is an LRTAdmin determined action, it is unlikely to happen, but has a important impact at the pool state.
## lo-05 getAssetBalances at the NodeDelegator loops through an externally determined array
[Reference](https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L102-L109)
The function is called by the NodeDelegator and can be a point of gas limit breach in case the strategies array become too big, leading to a DOS whenever this function is called inside a transaction.

```solidity
function getAssetBalances()
        external
        view
        override
        returns (address[] memory assets, uint256[] memory assetBalances)
    {
    ...
(IStrategy[] memory strategies,) =        IEigenStrategyManager(eigenlayerStrategyManagerAddress).getDeposits(address(this));
        uint256 strategiesLength = strategies.length;
        assets = new address[](strategiesLength);
        assetBalances = new uint256[](strategiesLength);
        for (uint256 i = 0; i < strategiesLength;) {
            assets[i] = address(IStrategy(strategies[i]).underlyingToken());
            assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
            unchecked {
                ++i;
            }
        }
 }
```

## lo-06 updatePriceFeedFor at the ChainlinkPriceOracle contract may utilize a non-price feed account
[Reference](https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L45-L49)
At the updatePriceFeedFor function, the ChainlinkPriceOracle contract does not check whether the target address is compliant with a price feed interface. This can lead to the contract utilizing invalid data feeds for it's prices.
```solidity
function updatePriceFeedFor(address asset, address priceFeed) external onlyLRTManager onlySupportedAsset(asset) {
        UtilLib.checkNonZeroAddress(priceFeed);
        assetPriceFeed[asset] = priceFeed;
        emit AssetPriceFeedUpdate(asset, priceFeed);
    }

```

## typo-01 - qa-119
[Reference](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L40)
The comment refers to different tokens
```solidity
/// @param admin Admin address
/// @param stETH stETH address
/// @param rETH rETH address
/// @param cbETH cbETH address
/// @param rsETH_ cbETH address
```
While it should refer to rsETH.
