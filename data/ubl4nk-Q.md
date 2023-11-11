### Return value of `depositIntoStrategy` not checked
From the docs of depositIntoStrategy:
> @return shares The amount of new shares in the `strategy` created as part of the action.

But the returned value is not checked [at this line](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/NodeDelegator.sol#L67):
```solidity
IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
```
Make sure the asset is successfully deposited into strategy by checking the return value depositIntoStrategy.
