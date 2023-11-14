# QA report for Kelp DAO contest by Catellatech

## [QA-01] onlyLRTManager can approve the max asset in NodeDelegator::maxApproveToEigenStrategyManager even when the contract is paused 
Consider add the `whenNotPaused` modifier to the function `NodeDelegator::maxApproveToEigenStrategyManager` as well.

- https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L38C1-L46C6

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
## [QA-02] Remove/resolve the comments like the following
in `LRTDepositPool::getAssetDistributionData` in line [78](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L78) the following comment it was not remove ` // Question: is here the right place to have this? Could it be in LRTConfig?`. Please remove or resolve that comment.

## [QA-03] in the `LRTConfig.sol` contract the team did not consider adding a function to remove assets
Context: The contract handles `addNewSupportedAsset` and `updateAssetStrategy` but does not account for a scenario where they have to remove an asset from the LRTConfig; in that case, it won't be possible.

## [QA-04] Inconsistent Use of NatSpec
NatSpec is a boon to all Solidity developers. Not only does it provide a structure for developers to document their code within the code itself, it encourages the practice of documenting code. When future developers read code documented with NatSpec, they are able to increase their capacity to understand, upgrade, and fix code. Without code documented with NatSpec, that capacity is hindered.

The Centrifuge codebase does have a high level of documentation with NatSpec. However there are numerous instances of functions missing NatSpec.

### Possible Risks
1. Lack of understanding of code without adequate documentation.
2. Difficulty in understanding, upgrading, and fixing code without documentation.

### Recommended Mitigation Steps
- Practice consistent use of NatSpec on all parts of the project codebase.
- Include the need for NatSpec comments for code reviews to pass.

## [QA-05]  Crucial information is missing on important functions in all contracts
In the constructor functions it is not specified in the documentation if the admin/roles will be an EOA or a contract. Consider improving the docstrings to reflect the exact intended behaviour, and using `Address.isContract` function from OpenZeppelin’s library to detect if an address is effectively a contract.

