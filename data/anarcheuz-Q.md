LRTOracle.sol has a pausing functionality: https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L102. However, it is not being used anywhere in the code.

It is recommended to remove unused code to reduce gas usage and avoid introducing needless complexity. 