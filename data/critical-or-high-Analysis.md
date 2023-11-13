rsETH protocol allows users to re-stake their LSTs (currently, it supports only stETH, rETH, and cbETH) on the Eigen Layer and accrue rewards from the EigenLayer. This is the economic model of rsETH - users get rewarded over time for locking their LSTs.

In the Eigen Layer, LSTs are staked in so-called [strategies](https://github.com/Layr-Labs/eigenlayer-contracts/blob/master/src/contracts/core/StrategyManager.sol). The contract `LTRDepositPool` stakes LSTs through `NodeDelegator` contracts, which communicate subsequently with `StrategyManager` on the EigenLayer.

Currently, rsETH protocol lacks several crucial pieces:
 1. It's not possible to redeem/withdraw rsETH back to LSTs with a reward. Therefore breaking the economic model of the protocol.
 2. `NodeDelegator` only deposits to a strategy on the EigenLayer. However, it's not enough. To accrue interest on the EigenLayer, staker should [delegate](https://github.com/Layr-Labs/eigenlayer-contracts/blob/master/docs/core/DelegationManager.md#delegateto) their shares to an operator or [create a new operator](https://github.com/Layr-Labs/eigenlayer-contracts/blob/master/docs/core/DelegationManager.md#registerasoperator) for themselves.
 3. `NodeDelegator` isn't able to [withdraw](https://github.com/Layr-Labs/eigenlayer-contracts/blob/master/docs/core/DelegationManager.md#queuewithdrawal) from the EigenLayer. So, funds are stuck there permanetly.
 4.  rsETH protocol should use battle-tested [ERC4626](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC4626.sol) for the `LTRDepositPool`.
 
 Manual review approach has been taken for the audit.

### Time spent:
16 hours