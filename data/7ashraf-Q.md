# Check if deposit limit is valid 
## Instances
* LRTConfig.sol #80
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L80

* LRTConfig.sol #102
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L102

#  Division by zero risk
## Instances 
LRTDepositpool.sol #109
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L109C28-L109C29
## Mititgation
An oracle is high-likely to return a zero price value, just to be safe add ```require value >0``` block.

# Use safeTransferFrom 
## Instances 
LRTDepositpool.sol #136
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L136


# Missing not paused modifier 
## Instances
* LRTDepositpool.sol #183
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L183

* LRTDepositpool.sol #202
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L202

* NodeDelegator.sol #38
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L38 

# Check for minted amount > 0
## Instances
* LRTDepositpool.sol #141
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L141

* LRTDepositpool.sol #152
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L152
## Description
Should check the returned value after calling mint to be greater than 0 to make sure the protocol is working properly



