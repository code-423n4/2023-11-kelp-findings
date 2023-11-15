# [L-1] Inadequate Validation of Chainlink Oracle Price Data

The bot reports in [L-02] Avoid the use of deprecated Chainlink functions recommend using a new function to get the latest price. However even when using the `latestRoundData` function, the results should be validated, however it does not mention. 

The returned value should be validated and a threshold should be set when the prices should be updated to avoid stale price.


# [L-2] Deposit will revert

In each strategy, especially `StrategyBaseTVLLimits`, it has the `maxPerDeposit` and `maxTotalDeposits`.

        /// The maximum deposit (in underlyingToken) that this strategy will accept per deposit
        uint256 public maxPerDeposit;

        /// The maximum deposits (in underlyingToken) that this strategy will hold
        uint256 public maxTotalDeposits;
https://github.com/Layr-Labs/eigenlayer-contracts/blob/75e59432d079c6f90d48d4e950a68c15867c82b2/src/contracts/strategies/StrategyBaseTVLLimits.sol#L14-L18

If the balance in NodeDelegator is more than one of two numbers above, the deposit will be reverted. Because NodeDelegator will send the whole balance to the Strategy.

        uint256 balance = token.balanceOf(address(this));

        emit AssetDepositIntoStrategy(asset, strategy, balance);

        IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);

https://github.com/code-423n4/2023-11-kelp/blob/ee1154fcb6f6619cdc9aeda27503d9a2cbf6d8eb/src/NodeDelegator.sol#L63-L67