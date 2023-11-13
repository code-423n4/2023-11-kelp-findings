1.
Defining DEFAULT_ADMIN_ROLE should be moved from:
https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConfigRoleChecker.sol#L28

to LRTConstants.sol for more consistency and mitigating bugs.

2.
onlySupportedAsset modifier exists in two places.
Changing one in future can lead to bugs if forgetting the change the other one.
Recommendation: To remain only in LRTConfig.sol, as the RoleChecker contract is more for role things.