## [L-01] `whenNotPaused` modifier is not used
### Description
Contract does not use `whenNotPaused` anywhere while it inherits `PausableUpgradeable` library.
It is noted that this issue is covered in Bot-Race but there is only one reference provided. Here I am attaching all references.
### References
https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/LRTDepositPool.sol
https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/LRTOracle.sol
https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/NodeDelegator.sol