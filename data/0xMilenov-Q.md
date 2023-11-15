## Heading
Role mismatch in addNodeDelegatorContractToQueue function

## Context
[LRTDepositPool.sol](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTDepositPool.sol#L191-L211)

## Impact
The addNodeDelegatorContractToQueue function's control by onlyLRTAdmin, contrary to its NatSpec comment indicating onlyLRTManager access, may lead to administrative overreach and operational inefficiencies.

## Description
At addNodeDelegatorContractToQueue function has a role mismatch. It is currently restricted to LRT Admins but is documented as being accessible only by LRT Managers. This discrepancy can lead to administrative errors and potential security issues, as the admin role might not be intended for operational tasks like updating node delegator contracts.

```solidity
/// @notice add new node delegator contract addresses
    /// @dev only callable by LRT manager
    /// @param nodeDelegatorContracts Array of NodeDelegator contract addresses
    function addNodeDelegatorContractToQueue(
        address[] calldata nodeDelegatorContracts
    ) external onlyLRTAdmin {
        // @audit onlyLRTManager!
        uint256 length = nodeDelegatorContracts.length;
        if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
            revert MaximumNodeDelegatorCountReached();
        }

        for (uint256 i; i < length; ) {
            UtilLib.checkNonZeroAddress(nodeDelegatorContracts[i]);
            nodeDelegatorQueue.push(nodeDelegatorContracts[i]);
            emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]);
            unchecked {
                ++i;
            }
        }
    }
```

## Recommendation
Change the modifier to onlyLRTManager.