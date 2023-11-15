## Non-Usage of safeTransfer function

The smart contracts currently utilize standard transfer methods for handling liquid staking tokens instead of the safeTransfer method. While currently functional due to the nature of stETH, rETH, and cbETH (which all return true on transfer operations), this approach poses potential risks for future integrations with other staking tokens.

## Unbounded number of supported tokens can lead to a DoS scenario

The LRTConfig lacks an upper limit on supported tokens. This causes an issue in the getRSETHPrice function within the LRTOracle, where an unbounded loop occurs. With a sufficiently large number of assets, this loop can consume excessive gas, making the contract inoperable. Recovery requires administrative intervention to replace the protocol configuration.

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L80

## Components might disagree on protocol configurations

The LRTDepositPool, LRTOracle, and NodeDeligator smart contract components currently use individual protocol configurations. It is essential for the integrity and consistent operation of the system that these configurations reference the same address. The absence of a unified configuration address can result in unexpected and potentially disruptive behavior. For example, obtaining prices for certain assets becomes impossible if the LRTDepositPool and LRTOracle disagree on the set of supported tokens due to separate configurations. This property is currently not enforced on-chain.

## Various state-changing functions to not emit events

Several functions do not emit events after modifying the internal state. This omission hinders stakeholders' ability to monitor protocol parameters effectively. Examples include:
https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L109 and
https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L144
