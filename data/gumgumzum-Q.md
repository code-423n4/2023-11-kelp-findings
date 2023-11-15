### NodeDelegator deposits possible failure
In [NodeDelegator@depositAssetIntoStrategy](https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L51-L68), when the deposited LST balance is close to the remaining available deposits (`maxTotalDeposits - depositedSoFar`) for the corresponding EigenLayer Strategy, the deposit will fail if it is front ran by other regular deposits or a malicious user sending enough LST tokens directly to the NodeDelegator to tip it over the limit.

This would require the admin to transfer enough LSTs back from the `NodeDelegator` to the `LRTDepositPool` to go back below the limit after each failure before reinitiating another.

### Potential revenue sharing issues
This is mainly based on the assumption that revenue sharing will be based on the rsETH amounts held by each user and the fact that :
* The three step process for depositing assets into strategies (User -> LRTDepositPool -> NodeDelegators -> EigenLayer Strategies).
* EigenLayer Strategies deposits limits means that the NodeDelegators will likely not be able to deposit all LSTs deposited by users.
* There is no way to differentiate which LSTs from which users were deposited to the EigenLayer Strategies.
* NodeDelegators will only claim rewards based on the amount of LSTs they deposited.

This means that any rewards claimed by NodeDelegators might be distributed across all rsETH holders regardless of whether their deposited LST tokens were forwarded to the EigenLayer Strategies or not.

### Oracle usage issues

For reference :
|    Pair     | Deviation | Heartbeat | Decimals |
|-------------|-----------|-----------|----------|
| RETH / ETH  | 2%        | 86400s    | 18       |
| CBETH / ETH | 1%        | 86400s    | 18       |
| STETH / ETH | 0.5%      | 86400s    | 18       |

* Using [latestAnswer](https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L38) instead of `latestRoundData` with sanity checks (.e.g for stale price,
)
* Considering rsETH can be tradable and the absence of fees when depositing, arbitrage opportunities (Flashloan LST / swap ETH to LST -> deposit LST -> swap rsETH to ETH / LST and repay flashloan) may arise when the price of an LST fluctuates within the deviation and heartbeat of the associated oracle.
* Decimals are assumed to be 18 and while this is true for the currently supported assets oracles, nothing guarantees that future ones will abide to that.
* A revert from one / multiple of the supported LSTs oracles can lead to a full / partial DoS of deposits and any other parts relying on the LST prices.
* A failover is not possible without admin action as only [one](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L97) oracle can be used per asset at once

### Since LST can only use [one strategy](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L109-L122), deposit limits should be lower than the corresponding EigenLayer Strategies limits (currently 100k as well for [stETH](https://etherscan.io/address/0x93c4b944D05dfe6df7645A86cd2206016c51564D#readProxyContract#F2), [cbETH](https://etherscan.io/address/0x54945180dB7943c0ed0FEE7EdaB2Bd24620256bc#readProxyContract#F2) and [rETH](https://etherscan.io/address/0x1BeE69b7dFFfA4E2d53C2a2Df135C388AD25dCD2#readProxyContract#F2) strategies)

### Since LRTDepositPool, LRTOracle, RSETH and EigenLayer StrategyManager are upgradable contracts, their addresses can be immutable once set in [LRTConfig](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol)

