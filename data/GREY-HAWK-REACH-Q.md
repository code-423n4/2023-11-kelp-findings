# Lines of code

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L132-L134

## Impact
Before users can deposit funds into the ```LRTDepositPool.sol``` contract via the ```depositAsset()``` function, a check is performed to see if the current amount doesn't exceed the maxPerDeposit set by the EigenLyaer strategy.
The EigenLayer strategy contract [StrategyBaseTVLLimits.sol](https://github.com/Layr-Labs/eigenlayer-contracts/blob/master/src/contracts/strategies/StrategyBaseTVLLimits.sol) where funds will be deposited has two important checks 
```
/// The maximum deposit (in underlyingToken) that this strategy will accept per deposit
    uint256 public maxPerDeposit;
```
and

```
/// The maximum deposits (in underlyingToken) that this strategy will hold
    uint256 public maxTotalDeposits;
```
The ```depositLimitByAsset``` set in the ```LRTConfig.sol``` contract is there to check if the ```maxPerDeposit``` set in the Eigenlayer protocol is not exceeded. Problem arises in how this value is checked. As can be seen in the POC  ```getAssetBalance()``` returns deposited tokens + accrued rewards. Depending on how often deposits on EigenLayer strategies are not paused Kelp may not gather all of the allowed amount for a deposit. Which results in loosing possible rewards. As each deposit is performed there will be more accrued rewards resulting in a smaller pot gathered by depositors for a stake in EigenLayer strategies.

## Proof of Concept
  **1.** When kelp first start gathering deposit the hardcoded limit is 100_000e18, if this funds are collected and deposited now kelp have a deposit into EigenLayer which is generating yield
  **2.** Then kelp will increase the ```depositLimitByAsset``` to 200_000e18 in order to gather new deposits from users, and deposit in a for example a month when eigen unpauses their deposit function.
  **3.** During this time kelp has already deposited 100_000e18 tokens that will be generating yield
  **4.** Instead of gathering 100_000e18 for a new deposit they will gather lets say 98_000e18
  **5.** Which results in 2_000e18 lost deposits that can be generating yield for the next month
Keep in mind that the probability of ```maxPerDeposit``` being set to 100_000e18 on mainet is low, as the first time EigenLayer opened up their deposits the maxTotalDeposit for strategies was 3600e18.

 - In order to deposit funds the user calls ```depositAsset()``` function in the ```LRTDepositPool.sol``` contract
```
 function depositAsset(
        address asset,
        uint256 depositAmount
    )
        external
        whenNotPaused
        nonReentrant
        onlySupportedAsset(asset)
    {
        // checks
        if (depositAmount == 0) {
            revert InvalidAmount();
        }
        if (depositAmount > getAssetCurrentLimit(asset)) {
            revert MaximumDepositLimitReached();
        }

        if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) {
            revert TokenTransferFailed();
        }

        // interactions
        uint256 rsethAmountMinted = _mintRsETH(asset, depositAmount);

        emit AssetDeposit(asset, depositAmount, rsethAmountMinted);
    }
```
 - ```depositAsset()``` in turn calls ```getAssetCurrentLimit()``` which first calls ```depositLimitByAsset()``` in order to take the governance limit set for the specified asset

``` 
function getAssetCurrentLimit(address asset) public view override returns (uint256) {
        return lrtConfig.depositLimitByAsset(asset) - getTotalAssetDeposits(asset);
    }
```
 - Then substracts the amount returned from ```getTotalAssetDeposits()``` 

```
function getTotalAssetDeposits(address asset) public view override returns (uint256 totalAssetDeposit) {
        (uint256 assetLyingInDepositPool, uint256 assetLyingInNDCs, uint256 assetStakedInEigenLayer) =
            getAssetDistributionData(asset);
        return (assetLyingInDepositPool + assetLyingInNDCs + assetStakedInEigenLayer);
    }
```

 - ```getTotalAssetDeposits()``` in turn calls ```getAssetDistributionData()```

```
    function getAssetDistributionData(address asset)
        public
        view
        override
        onlySupportedAsset(asset)
        returns (uint256 assetLyingInDepositPool, uint256 assetLyingInNDCs, uint256 assetStakedInEigenLayer)
    {
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
    }
```

 - ```getAssetDistributionData()``` check the balance of ```LRTDepositPool.sol``` for the selected token  and the balance of ```NodeDelegator.sol``` for the selected token then calls ```getAssetBalance()``` in the ```NodeDelegator.sol```

```
function getAssetBalance(address asset) external view override returns (uint256) {
        address strategy = lrtConfig.assetStrategy(asset);
        return IStrategy(strategy).userUnderlyingView(address(this));
    }
```
 - ```getAssetBalance()``` calls [userUnderlyingView](https://github.com/Layr-Labs/eigenlayer-contracts/blob/master/src/contracts/strategies/StrategyBase.sol#L245-L251) on the EigenLayer protocol, 
 - which in turn calls  [sharesToUnderlyingView](https://github.com/Layr-Labs/eigenlayer-contracts/blob/master/src/contracts/strategies/StrategyBase.sol#L200-L206) which converts the number of shares to the equivalent amount of underlying tokens for the selected strategy meaning. Meaning that this will return already deposited tokens + accrued rewards.
