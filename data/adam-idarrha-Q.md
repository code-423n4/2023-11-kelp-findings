Title: depositors might not be able to deposit even if deposit limit is not reached

## Lines of code:

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/oracles/ChainlinkPriceOracle.sol#L37-L39

## Vulnerability details:
### Details:

the protocol has a deposit limit per asset for all depositor calculated in the following way:
total assets in LRTDepositPool + total assets in nodeDelegators + amountDepositedInEigenLayer: 

```solidity=47
function getTotalAssetDeposits(address asset) public view override returns (uint256 totalAssetDeposit) {
        (uint256 assetLyingInDepositPool, uint256 assetLyingInNDCs, uint256 assetStakedInEigenLayer) =
            getAssetDistributionData(asset);
        return (assetLyingInDepositPool + assetLyingInNDCs + assetStakedInEigenLayer);
    }
```

the amount Deposited in EigenLayer is calculated with the function `userUnderlyingView` which returns how much asset the user has in the pool, based on how many shares he has. which takes into account amount deposited + any rewards accrued from the strategy.

`NodeDelegator`

```solidity=121
function getAssetBalance(address asset) external view override returns (uint256) {
        address strategy = lrtConfig.assetStrategy(asset);
        return IStrategy(strategy).userUnderlyingView(address(this));
    }
```

`StrategyBase.sol` out of scope:

```solidity
/**
     * @notice convenience function for fetching the current underlying value of all of the `user`'s shares in
     * this strategy. In contrast to `userUnderlying`, this function guarantees no state modifications
     */
    function userUnderlyingView(address user) external view virtual returns (uint256) {
        return sharesToUnderlyingView(shares(user));
    }
```

so the amount deposited in eigenlayer returns how much the user deposited + any rewards or slashing.

which makes it that the protocol think that he reached the deposit limit without it actually reaching it.

```solidity=56
function getAssetCurrentLimit(address asset) public view override returns (uint256) {
        return lrtConfig.depositLimitByAsset(asset) - getTotalAssetDeposits(asset);
    }
```

### Tools Used:

vscode

### Recommended Mitigation Steps:

keep track of much is deposited for each asset in the `LRTDepositPool`. and use that to see if it's surpasses the limit.