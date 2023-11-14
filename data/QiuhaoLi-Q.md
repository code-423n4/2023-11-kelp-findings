## [LOW] LRTConfigRoleChecker should add a storage gap

`LRTConfigRoleChecker` is a contract inherited by `LRTDepositPool`, `LRTOracle`, `NodeDelegator`, `RSETH`, etc, which are upgradable contracts. E.g. for `LRTDepositPool`, it's defined as:

```solidity
contract LRTDepositPool is ILRTDepositPool, LRTConfigRoleChecker, PausableUpgradeable, ReentrancyGuardUpgradeable {
``` 

In `LRTConfigRoleChecker`, we have a storage variable, but don't have storage gap:

```solidity
abstract contract LRTConfigRoleChecker {
    ILRTConfig public lrtConfig;
......
}
```

To avoid future needs to add storage slot in `LRTConfigRoleChecker` and storage conflicts. Better add a storage gap:

```solidity
    **
     * @dev This empty reserved space is put in place to allow future versions to add new
     * variables without shifting down storage in the inheritance chain.
     * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
     *
    uint256[49] private __gap;
```