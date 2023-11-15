1.Unnecessary import of PausableUpgradeable in the LRTOracle contract should be removed
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L102#L109

2.ndcIndex not checked may led to Index Out Of bounds issue
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L183#L197
