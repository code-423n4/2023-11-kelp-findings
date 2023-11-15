```
function maxApproveToEigenStrategyManager(address asset)
function depositAssetIntoStrategy(address asset)
function transferBackToLRTDepositPool(
      address asset,
      uint256 amount
)
```

Alle the functions above rely on lrtConfig.getContract(CONSTANT) which may return a 0 address if it has not yet been set. This might cause approve/depositIntoStrategy/transfer to this 0 address.

E.g.
```
IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);

IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);

if (!IERC20(asset).transfer(lrtDepositPool, amount)) {
     revert TokenTransferFailed();
}
```

Additional, checks and reverting with a CustomError could reduce human-produced errors in this case.