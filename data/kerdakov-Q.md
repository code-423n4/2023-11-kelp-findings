The use of `unchecked` in the for loop in [NodeDelegator](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/NodeDelegator.sol#L109C8-L115C10) seems unnecessary and could potentially lead to unintended consequences. The loop itself doesn't involve arithmetic operations that could overflow or underflow, so using `unchecked` here is unnecessary.

```
for (uint256 i = 0; i < strategiesLength;) {
    assets[i] = address(IStrategy(strategies[i]).underlyingToken());
    assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
    unchecked {
        ++i;
    }
}
```

In fact, it's more common and safer to use a standard for loop without unchecked:

```
for (uint256 i = 0; i < strategiesLength; i++) {
    assets[i] = address(IStrategy(strategies[i]).underlyingToken());
    assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
}
```