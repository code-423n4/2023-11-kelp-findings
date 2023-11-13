## Impact
onlySupportedAsset modifier exists in two places and at some point a small change in one of the modifiers can lead to forgetting to change it in the other one -> Bugs.

## Proof of Concept

## Tools Used
Manual analysis

## Recommended Mitigation Steps
For me it looks like this should only exist in LRTConfig.sol, because the RoleChecker contract is more for roles.