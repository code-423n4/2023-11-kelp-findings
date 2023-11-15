# Findings Summary

| ID            | Title                                                                                                                               | Severity |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------- | -------- |
| [L-01](#l-01) | `NodeDelegator` contract doesn't have a mechanism to remove LST asset approval from `eigenStrategyManager` contract                 | Low      |
| [L-02](#l-02) | `LRTConfig.setRSETH` function: updating the `rsETH` contract address will result in loss of users deposits                          | Low      |
| [L-03](#l-03) | `LRTConfig` contract: no mechanism implemented to remove supported LST assets                                                       | Low      |
| [L-04](#l-04) | `LRTDepositPool.addNodeDelegatorContractToQueue` function is accessed by the LRTAdmin while it should be accessed by the LRTManager | Low      |
| [L-05](#l-05) | `LRTDepositPool.transferAssetToNodeDelegator` function doesn't check if the `nodeDelegator` exists before sending it LST assets     | Low      |

# Low

## [L-01] `NodeDelegator` contract doesn't have a mechanism to remove LST asset approval from `eigenStrategyManager` contract <a id="l-01" ></a>

## Impact

- The main purpose of the node delegators is to act as temporary LST holders before depositing in EigenLayer protocol (3rd party); mainly to prevent disabling the `kelp` protocol if the EigenLayer paused the deposit operations.

- First the LST assets are transferred to a `nodeDelegator` contract by the deposit pool manager, then transferred to `eigenStrategyManager` contract by calling `NodeDelegator.depositAssetIntoStrategy` function that will in turn calls the `IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);`, but before that, the nodeDelegator should approve the `eigenStrategyManager` contract on the deposited amount.

- And for this approval action; the `nodeDelegator` contract manager calls `NodeDelegator.maxApproveToEigenStrategyManager` function, which will approve the strategy manager on the maximum LST amount.

- But if the `eigenStrategyManager` contract got compromised/hacked or started to act maliciously, the LST asset balance of the `nodeDelegator` contract will be at risk since there's no way to remove LST assets approvals for the comromised strategy manager contract (setting the LST approval/allowance to zero) in cases of emergencies.

## Proof of Concept

[NodeDelegator.maxApproveToEigenStrategyManager function](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L38-L46)

```solidity
   function maxApproveToEigenStrategyManager(address asset)
        external
        override
        onlySupportedAsset(asset)
        onlyLRTManager
    {
        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);
        IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);
    }
```

## Tools Used

Manual Review.

## Recommended Mitigation Steps

Remove `NodeDelegator.maxApproveToEigenStrategyManager` function, and update `NodeDelegator.depositAssetIntoStrategy` function to approve the strategy manager contract on the deposited amount only before depositing in the strategy:

```diff
   function depositAssetIntoStrategy(address asset)
        external
        override
        whenNotPaused
        nonReentrant
        onlySupportedAsset(asset)
        onlyLRTManager
    {
        address strategy = lrtConfig.assetStrategy(asset);
        IERC20 token = IERC20(asset);
        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

        uint256 balance = token.balanceOf(address(this));

        emit AssetDepositIntoStrategy(asset, strategy, balance);
+       IERC20(asset).approve(eigenlayerStrategyManagerAddress, balance);
        IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
    }
```

```diff
-   function maxApproveToEigenStrategyManager(address asset)
-        external
-        override
-        onlySupportedAsset(asset)
-        onlyLRTManager
-    {
-        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);
-        IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);
-    }
```

## [L-02] `LRTConfig.setRSETH` function: updating the `rsETH` contract address will result in loss of users deposits <a id="l-02" ></a>

## Impact

- The admin of the `LRTConfig` can change the address of the `rsETH` contract, and this will result in re-setting the `rsETH.totalSupply` to zero.

- So if the `rsETH` contract is updated; users can't redeem/withdraw their deposits as the `totalSupply` would be zero (would revert due to underflow if any value is deducted from zero), also the price of the `rsETH` share token will drop as it's calculated based on the `totalSupply` and the total value of ETH held by the protocol (deposited in the pool + in the node delegators + in the EigenLayer strategies):

  ```solidity
  return totalETHInPool / rsEthSupply;
  ```

## Proof of Concept

[LRTConfig.setRSETH function](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L144-L147)

```solidity
    function setRSETH(address rsETH_) external onlyRole(DEFAULT_ADMIN_ROLE) {
        UtilLib.checkNonZeroAddress(rsETH_);
        rsETH = rsETH_;
    }
```

## Tools Used

Manual Review.

## Recommended Mitigation Steps

Either prevent updating the `stETH` contract address or implement a mechanism to transfer users balances and totalSupply with the new contract.

# --------------------------------

## [L-03] `LRTConfig` contract: no mechanism implemented to remove supported LST assets <a id="l-03" ></a>

## Impact

- The `LRTConfig` contract has a function to add LST supported assets to the protocol, but it doesn't have any function or mechanism to unsupport any of these assets.

- So if the protocol decided to remove any LST asset from being traded in the system due to financial risks, it will not be able to do so and users will keep depositing these assets and receive share tokens in return.

## Proof of Concept

[LRTConfig.\_addNewSupportedAsset function](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L80-L89)

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

## Tools Used

Manual Review.

## Recommended Mitigation Steps

Add a mechanism to remove/unsupport any of the added LST assets.

## [L-04] `LRTDepositPool.addNodeDelegatorContractToQueue` function is accessed by the LRTAdmin while it should be accessed by the LRTManager <a id="l-04" ></a>

## Impact

- `addNodeDelegatorContractToQueue` function is used to add `nodeDelegator` contracts to the system to act as a LST hoders until depositing these assets in EigenLayer protocol, as per the comments on the `addNodeDelegatorContractToQueue` function, it should be called only by the LRTManager:

  >     /// @dev only callable by LRT manager

- But the function has `onlyLRTAdmin` access modifier that only allows the admin from accessing it,; not the manager.

## Proof of Concept

[LRTDepositPool.addNodeDelegatorContractToQueue function/L159-L162](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L159-L162)

```solidity
    /// @notice add new node delegator contract addresses
    /// @dev only callable by LRT manager
    /// @param nodeDelegatorContracts Array of NodeDelegator contract addresses
    function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {
```

## Tools Used

Manual Review.

## Recommended Mitigation Steps

```diff
-   function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {

+   function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTManager {
```

## [L-05] `LRTDepositPool.transferAssetToNodeDelegator` function doesn't check if the `nodeDelegator` exists before sending it LST assets <a id="l-05" ></a>

## Impact

- The main purpose of node delegators is to act as temporary LST holders before depositing in EigenLayer protocol (3rd party); mainly to prevent disabling the `kelp` protocol if the EigenLayer paused the deposit operations.

- Whenever the deposit limit of a specific LST asset is reached, a new `nodeDelegator` contract is deployed and added to the protocol to overcome the deposit limit implemented by EigenLayer protocol.

- `transferAssetToNodeDelegator` function is intended to be called by the deposit pool manager to transfer LST assets to an approved `nodeDelegator` contract; and then these assets to be moved to EigenLayer strategy when the strategy is not paused.

- but this function doesn't check if the `ndcIndex` (the index of the `nodeDelegator` in the `nodeDelegatorQueue` array) is within the `nodeDelegatorQueue` array or not; which might lead to LST assets being transferred to `address(0)` if the `nodeDelegator` doesn't exist (`ndcIndex` is out of bounds).

- This will lead to loss of protocol funds and affecting the price of the `rsETH` share tokens as the total ETH locked in the protocol will be lowered without burning an equivalenT amount of shares (the price of `rsETH` will go up!).

## Proof of Concept

[LRTDepositPool.transferAssetToNodeDelegator function](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L183-L197)

```solidity
    function transferAssetToNodeDelegator(
        uint256 ndcIndex,
        address asset,
        uint256 amount
    )
        external
        nonReentrant
        onlyLRTManager
        onlySupportedAsset(asset)
    {
        address nodeDelegator = nodeDelegatorQueue[ndcIndex];
        if (!IERC20(asset).transfer(nodeDelegator, amount)) {
            revert TokenTransferFailed();
        }
    }
```

## Tools Used

Manual Review.

## Recommended Mitigation Steps

Update `transferAssetToNodeDelegator` function to check if the `nodeDelegator` address exists bfore transferring LST assets to it:

```diff
    function transferAssetToNodeDelegator(
        uint256 ndcIndex,
        address asset,
        uint256 amount
    )
        external
        nonReentrant
        onlyLRTManager
        onlySupportedAsset(asset)
    {
        address nodeDelegator = nodeDelegatorQueue[ndcIndex];
+       require(nodeDelegator != address(0),"nodeDelegator doesn't exist");
        if (!IERC20(asset).transfer(nodeDelegator, amount)) {
            revert TokenTransferFailed();
        }
    }
```
