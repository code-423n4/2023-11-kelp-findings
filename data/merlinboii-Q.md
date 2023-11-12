## Title: Inconsistency Between comments and functionality

## Related Files 
* https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L160
* https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L162

## Impact
The inconsistency between the comment and the actual functionality **could lead to confusion for developers and users.**

# Vulnerabilities 
The `addNodeDelegatorContractToQueue` function of the `LRTDepositPool` contract is **restricted to only being callable by the LRT admin (L162).** However, the comment associated with this function incorrectly states that it is restricted to **only being callable by the LRT manager (L160)**, creating confusion about the actual access control.

## Recommendation 
Consider updating the comment (L160) to accurately reflect the functionality, specifying that the `addNodeDelegatorContractToQueue` function is restricted to only being callable by the LRT admin.