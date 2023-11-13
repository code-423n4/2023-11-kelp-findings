## [G-01] Avoid explicitly initializing variables to default values
### Description
Explicitly initializing a variable with its default value, such as 0 for uint, false for bool, or address(0) for address when it's not set/initialized, is considered an antipattern and results in unnecessary gas consumption.
### Affected Line
https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/NodeDelegator.sol#L109

## [G-02] x = x + y is cheaper than x += y
### Affected Line
https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/LRTDepositPool.sol#L83
https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/LRTDepositPool.sol#L84
https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/LRTOracle.sol#L71