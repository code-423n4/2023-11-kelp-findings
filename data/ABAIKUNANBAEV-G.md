### Gas Optimizations List

| Number | Optimization Details                                                                                                     | Instances |
| :----: | :----------------------------------------------------------------------------------------------------------------------- | :-------: |
| [G-01] | Consider not using `nonReentrant` modifier where the possibility of reentrancy is 0.                  |     4    |
| [G-02] | Use hard-coded address instead of `address(this)` to save gas.                  |     6  |
| [G-03] | Consider not using pausability in the contract where `whenNotPaused` modifier is not used.               |  1 |
| [G-04] | Use assembly to implement for-loops.               |  6 |
| [G-05] | Use assembly to implement for address(0).               |  3 |

Total 5 issues.

## [G‑01] Consider not using `nonReentrant` modifier where the possibility of reentrancy is 0.   

Although it's considered as the best practice, it's recommended to use it with the functions with callbacks are possible such as ERC1155 or ERC777 functionality, for example:

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L125
```
 function depositAsset(
        address asset,
        uint256 depositAmount
    )
        external
        whenNotPaused
        nonReentrant
        onlySupportedAsset(asset)
    }
```

## [G‑02] Use hard-coded address instead of address(this) to save gas.  

In terms of gas, it's better to use pre-calculated contract address instead of `address(this)`:

```
assetLyingInDepositPool = IERC20(asset).balanceOf(address(this));
```

## [G‑03] Consider not using pausability in the contract where `whenNotPaused` modifier is not used. 

`LRTOracle.sol` `pause() and `unpause()` functions are initialized but `whenNotPaused` modifier is never used. Therefore, these 2 functions and pausability feature may not be implemented as it just consumes more gas and not actually represent any logic:

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L101-109
```
function pause() external onlyLRTManager {
        _pause();
    }

    /// @dev Returns to normal state. Contract must be paused
    function unpause() external onlyLRTAdmin {
        _unpause();
    }
}
```

## [G-04] Use assembly to implement for-loops.            

That's much cheaper to implement for-loops using assembly even though the readability suffers a bit.

```
 for (uint256 i = 0; i < strategiesLength;) {
            assets[i] = address(IStrategy(strategies[i]).underlyingToken());
            assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
            unchecked {
                ++i;
            }
```

## [G-05] Use assembly to check for address(0).

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L27
```
 UtilLib.checkNonZeroAddress(lrtConfigAddr);
```