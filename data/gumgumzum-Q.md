### Oracle usage

For reference :
|    Pair     | Deviation | Heartbeat | Decimals |
|-------------|-----------|-----------|----------|
| RETH / ETH  | 2%        | 86400s    | 18       |
| CBETH / ETH | 1%        | 86400s    | 18       |
| STETH / ETH | 0.5%      | 86400s    | 18       |

* Using [latestAnswer](https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L38) instead of `latestRoundData` with sanity checks (.e.g for stale price)
* Considering rsETH can be tradable and the absence of fees when depositing, arbitrage opportunities (Flashloan LST / swap ETH to LST -> deposit LST -> swap rsETH to ETH / LST and repay flashloan) may arise when the price of an LST fluctuates within the deviation and heartbeat of the associated oracle.
* Decimals are assumed to be 18 and while this is true for the currently supported assets oracles, nothing guarantees that future ones will abide to that.
* A revert from one / multiple of the supported LSTs oracles can lead to a full / partial DoS of deposits and any other parts relying on the LST prices.
* A failover is not possible without admin action as only [one](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L97) oracle can be used per asset at once

### Since LST can only use [one strategy](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L109-L122), deposit limits should be lower than the corresponding EigenLayer Strategies limits (currently 100k as well for [stETH](https://etherscan.io/address/0x93c4b944D05dfe6df7645A86cd2206016c51564D#readProxyContract#F2), [cbETH](https://etherscan.io/address/0x54945180dB7943c0ed0FEE7EdaB2Bd24620256bc#readProxyContract#F2) and [rETH](https://etherscan.io/address/0x1BeE69b7dFFfA4E2d53C2a2Df135C388AD25dCD2#readProxyContract#F2) strategies)

### Since LRTDepositPool, LRTOracle, RSETH and EigenLayer StrategyManager are upgradable contracts, their addresses can be immutable once set in [LRTConfig](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol)

