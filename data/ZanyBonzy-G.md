1. Use uint256(1)/uint256(2) instead of true/false to save gas for changes
Avoids a Gsset (20000 gas) when changing from false to true, after having been true in the past. 
```
mapping(address token => bool isSupported) public isSupportedAsset;
```
LRTConfig.sol [L15](https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/LRTConfig.sol#L15)