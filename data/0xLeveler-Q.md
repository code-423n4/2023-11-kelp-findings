## Title
`nodeDelegator` and `LRTDepositPoolContract` do not have any mechanism to remove node delegators from the queue when needed

## Proof of Concept
contract has a mechanism to add node delegators, but none to validate if they're valid contracts and remove them if they're added by mistake.

```solidity
function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {
        uint256 length = nodeDelegatorContracts.length;
        if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
            revert MaximumNodeDelegatorCountReached();
        }

        for (uint256 i; i < length;) {
            UtilLib.checkNonZeroAddress(nodeDelegatorContracts[i]);
            nodeDelegatorQueue.push(nodeDelegatorContracts[i]);
            emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]);
            unchecked {
                ++i;
            }
        }
    }
```

## Tools Used
Manual Review

## Recommended Mitigation Steps
Adding a mechanism to remove malicious or wrongly added node delegators.