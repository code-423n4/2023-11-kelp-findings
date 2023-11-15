In NodeDelegator contract, if usset the deposit strategy, the asset cannot be withdrawn from EigenLayer contract


https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L51

if the manager do not set the asset strategy in LRTConfig, the return strategy is 0x00 address
```    address strategy = lrtConfig.assetStrategy(asset);
```
so recommend add 0x00 check (UtilLib.checkNonZeroAddress() ) before deposit asset into eigenlayer pool .