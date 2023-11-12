## Gas Optimizations
| |Issue|Instances|
|-|:-|:-:|
| [GAS-01] | Zero address check is done twice on same value | 2 | 
| [GAS-02] | Remove unused pause functionality | 1 | 

### [GAS-01] Zero address check is done twice on same value
In `LRTConfig.sol`, the zero address checks are performed inside the private functions with `UtilLib.checkNonZeroAddress(val)`. However the initializer already performs these zero address checks on all the parameters, thus, these checks are performed twice.
Consider moving the following checks to their respective external functions so that the checks are not performed twice during the initialization.

*Instances (2)*:
```solidity
File: src/LRTConfig.sol
81:        UtilLib.checkNonZeroAddress(asset);

157:        UtilLib.checkNonZeroAddress(val);

```
https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L81
https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTConfig.sol#L157

### [GAS-02] Remove unused pause functionality
The `LRTOracle` contract inherits from `PausableUpgradeable` and implements a `pause()` and `unpause()` functions. However the "pausing" has no effect because the `whenNotPaused` modifier is never used.
Consider removing the `PausableUpgradeable` inherited contract to save gas.

*Instances (1)*:
```solidity
File: src/LRTOracle.sol
19: contract LRTOracle is ILRTOracle, LRTConfigRoleChecker, PausableUpgradeable {

102:     function pause() external onlyLRTManager { //@audit [GAS] Pause has no effect
        _pause();
    }

107:    function unpause() external onlyLRTAdmin {
        _unpause();
    }

```
https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTOracle.sol#L19
https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTOracle.sol#L102-L109
