# [N-01] Incorrect variable comment.
The comment should be "rsETH address" instead of "cbETH address".
```solidity
File: src/LRTConfig.sol

40:    /// @param rsETH_ cbETH address
```
[L40](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L40)

# [N-02] Use same role limit for update asset information.
Consider use ```onlyRole(LRTConstants.MANAGER)``` for ```updateAssetStrategy```. Because ```updateAssetDepositLimit``` limit to MANAGER role, and the two functions are both used to change asset's information.
```solidity
File: src/LRTConfig.sol
109:    function updateAssetStrategy(
110:        address asset,
111:        address strategy
112:    )
113:        external
114:        onlyRole(DEFAULT_ADMIN_ROLE)
```
[L109-L114](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L109-L114)