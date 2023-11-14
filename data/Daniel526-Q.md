## A. Defensive Check Absence Before Deposit in depositAssetIntoStrategy Function
[Link](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/NodeDelegator.sol#L51-L68)
The `depositAssetIntoStrategy` function retrieves the balance of the asset and directly proceeds to deposit into the strategy without checking if the balance is non-zero. Without this check, the function might execute unnecessary deposit operations, potentially leading to unexpected behavior or wasted gas.

```solidity
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

    // No check for a non-zero balance before proceeding
    uint256 balance = token.balanceOf(address(this));

    IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);

    emit AssetDepositIntoStrategy(asset, strategy, balance);
}
```
The absence of a check for a non-zero balance before proceeding with the deposit operation might result in unnecessary calls to `depositIntoStrategy`. This is suboptimal both in terms of gas efficiency and the potential for unintended consequences within the strategies.
## Impact:
The impact of this issue is primarily efficiency-related, with unnecessary gas consumption and potential unexpected behavior within the strategies due to redundant deposit calls with a zero balance.
## Mitigation:
To mitigate this issue, it is recommended to include a defensive check before the deposit operation to ensure that the balance is non-zero. This can be achieved by adding the following code snippet:

```solidity
// Check if the balance is non-zero before proceeding with the deposit
require(balance > 0, "Zero balance, nothing to deposit");
```
## B. Duplicate Node Delegator Contract Addresses in `nodeDelegatorQueue`
[Link](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L162-L176)
The `addNodeDelegatorContractToQueue` function iterates through an array of Node Delegator contract addresses (`nodeDelegatorContracts`) and adds each address to the `nodeDelegatorQueue` without checking whether the address already exists in the queue. Consequently, the same Node Delegator contract address can be added multiple times, leading to duplicate entries in the queue.
```solidity
function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {
    uint256 length = nodeDelegatorContracts.length;
    if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
        revert MaximumNodeDelegatorCountReached();
    }

    for (uint256 i; i < length;) {
        address newContract = nodeDelegatorContracts[i];
        UtilLib.checkNonZeroAddress(newContract);

        bool alreadyExists = false;
        for (uint256 j = 0; j < nodeDelegatorQueue.length; j++) {
            if (nodeDelegatorQueue[j] == newContract) {
                alreadyExists = true;
                break;
            }
        }

        if (!alreadyExists) {
            nodeDelegatorQueue.push(newContract);
            emit NodeDelegatorAddedinQueue(newContract);
        } else {
            revert DuplicateNodeDelegatorContract();
        }

        unchecked {
            ++i;
        }
    }
}
```
## Impact:
The impact of this issue is that the nodeDelegatorQueue can contain duplicate entries, potentially leading to incorrect behavior when interacting with Node Delegator contracts and complicating the management of the queue.

## Mitigation:
Include a check within the function to ensure that a Node Delegator contract address is not already present in the `nodeDelegatorQueue` before adding it. If the address already exists, revert with an appropriate error. Here's an example modification:
```solidity
// Inside the loop before adding the contract address to the queue
if (!alreadyExists) {
    nodeDelegatorQueue.push(newContract);
    emit NodeDelegatorAddedinQueue(newContract);
} else {
    revert DuplicateNodeDelegatorContract();
}
```
