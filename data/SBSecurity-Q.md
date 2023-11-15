### Low Risk

| Count | Title |
| --- | --- |
| [L-01](#l-01-no-removesupportedasset-function-in-the-lrtconfig) | No removeSupportedAsset function in the LRTConfig |
| [L-02](#l-02-cbeth-has-blacklisting) | cbETH has blacklisting |
| [L-03](#l-03-consider-use-stethuds-oracle) | Consider use stETH/UDS oracle |

| Total Low Risk Issues | 3 |
| --- | --- |

### Non-Critical

| Count | Title |
| --- | --- |
| [NC-01](#n01-missing-events-in-sensitive-functions) | Missing events in sensitive functions |
| [NC-02](#n02-nodedelegatoraddedinqueue-event-does-not-follow-camalcase) | NodeDelegatorAddedinQueue event does not follow CamalCase |
| [NC-03](#n03-istrategy-is-not-up-to-date-with-eigenlayeristrategy) | IStrategy is not up to date with EigenLayer::IStrategy |

| Total Non-Critical Issues | 3 |
| --- | --- |

## Low Risks

## [L-01] No removeSupportedAsset function in the LRTConfig

**Issue Description:** The contract provides a way to add new supported assets through the `addNewSupportedAsset()` function, but there is no corresponding function to remove supported assets.

Without the ability to remove supported assets, the contract may face challenges in adapting to changing circumstances, such as changes in the project's strategy, token ecosystem, or regulatory requirements.

**Recommendation:** Consider adding a function like **`removeSupportedAsset`** that allows the contract owner or another authorized role to remove an asset from the list of supported assets

## [L-02] cbETH has blacklisting

**Issue Description:** The blacklisting mechanism in the `CBETH` token introduces potential complications and risks across various stages of the transaction lifecycle, from initial deposits in the LRTDepositPool to activities within the NodeDelegator and Eigen Strategy. If the `LRTDepositPool` or any of the `NodeDelegator` is blacklisted, it implies that the associated tokens will become trapped in these contracts. 

## [L-03] Consider use stETH/UDS oracle

**Issue Description: The sponsor has confirmed their choice of Chainlink as an oracle to fetch prices.** Since all other LST price feeds are 18 decimal places, they will most likely use `stETH/ETH` price feeds. However, this feed has a long heartbeat and a 2% deviation threshold, which could lead to fund loss. The 24-hour heartbeat and 2% deviation threshold mean the price can move up to 2% or remain unchanged for 24 hours before triggering a price update. This could result in the on-chain price being significantly different from the true stETH price, leading to incorrectly calculated rsETH to mint.

**Recommendation:** 

Consider using the `stETH/USD` oracle instead, as it offers a 1-hour heartbeat and a 1% deviation threshold. To accommodate this change, also need to adjust the decimal calculation in `LRTOracle::getAssetPrice` to multiply by `18 ** (token.decimal - pricefeed.decimal)` or a similar factor. This adjustment ensures that the price of all tokens is returned with the same decimal precision.

## Non-Critical

## **[N‑01] Missing events in sensitive functions**

**Issue Description:** The following functions lack event emission:

```solidity
File: src/LRTDepositPool.sol

183: function transferAssetToNodeDelegator(
184:     uint256 ndcIndex,
185:     address asset,
186:     uint256 amount
187: )
188:     external
189:     nonReentrant
190:     onlyLRTManager
191:     onlySupportedAsset(asset)
192: {
193:     address nodeDelegator = nodeDelegatorQueue[ndcIndex];
194:     if (!IERC20(asset).transfer(nodeDelegator, amount)) {
195:         revert TokenTransferFailed();
196:     }

          // @audit missing event
197: }
```

[183](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L183)

```solidity
File: src/NodeDelegator.sol

38: function maxApproveToEigenStrategyManager(address asset)
39:     external
40:     override
41:     onlySupportedAsset(asset)
42:     onlyLRTManager
43: {
44:     address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);
45:     IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);

        // @audit missing event
46: }

74: function transferBackToLRTDepositPool(
75:     address asset,
76:     uint256 amount
77: )
78:     external
79:     whenNotPaused
80:     nonReentrant
81:     onlySupportedAsset(asset)
82:     onlyLRTManager
83: {
84:     address lrtDepositPool = lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL);
85: 
86:     if (!IERC20(asset).transfer(lrtDepositPool, amount)) {
87:         revert TokenTransferFailed();
88:     }
        // @audit missing event
89: }
```

[38](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L38), [74](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L74)

**Recommendation:**

Add the following event

transferAssetToNodeDelegator() →
`event AssetsTransferredSuccessfullyToNodeDelegator(address nodeDelegator, uint256 amount)`

maxApproveToEigenStrategyManager() → `event MaxApprovedToEigenStrategyManager()`

transferBackToLRTDepositPool() → 
`event AssetsTransferredSuccessfullyBackToLTRDepositPool(address lrtDepositPool, uint256 amount)`

## **[N‑02] NodeDelegatorAddedinQueue event does not follow CamalCase**

**Issue Description:** The event `NodeDelegatorAddedinQueue` is not compliant with CamelCase conventions.

**Recommendation:** Rename to `NodeDelegatorAddedInQueue`

## **[N‑03]** IStrategy is not up **to date with `EigenLayer::IStrategy`**

**Issue Description:** The protocol includes the `IStrategy`, which appears to be similar to the `EigenLayer`, but it uses an older version that lacks the newly added `shares(address user)` function and has some parameter renames.

**Recommendation:** Consider using the latest version from the EigenLayer repository: 

https://github.com/Layr-Labs/eigenlayer-contracts/blob/master/src/contracts/interfaces/IStrategy.sol