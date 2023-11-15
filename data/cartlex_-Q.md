| ID | Description | Severity |
| :-: | - | :-: |
| [L-01](#l-01-admin-can-use-renouncerole-for-self) | Admin can use `renounceRole` for self | Low |
| [L-02](#l-02-transferassettonodedelegator-function-can-be-used-when-contract-is-paused)| `transferAssetToNodeDelegator()` function can be used when contract is paused. | Low |
| [L-03](#l-03-burnfrom-function-burn-tokens-without-an-allowance)| `burnFrom` function burn tokens without an allowance. | Low |
| [NC-01](#nc-01-updateassetstrategy-doesnt-used-in-constructor)| `updateAssetStrategy()` doesn't used in constructor. | Non-critical |
| [NC-02](#nc-02-lack-of-check-in-getassetcurrentlimit)| Lack of check in `getAssetCurrentLimit()`. | Non-critical |


# [L-01] Admin can use `renounceRole` for self.

## Description
`LRTConfig.sol` and `RSETH.sol` contracts inherited from `AccessControlUpgradeable.sol`.

Admin can renounce role to self, for example, if there is only one account with `DEFAULT_ADMIN_ROLE` role, after admin renounce it, some important functionality can't be used.

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/RSETH.sol

## Recommended Mitigation Steps
Consider to override `renounceRole` so the admin can't renounce `DEFAULT_ADMIN_ROLE` for self.


# [L-02] `transferAssetToNodeDelegator()` function can be used when contract is paused.

## Description
Manager still can use `transferAssetToNodeDelegator()` function when `LRTDepositPool.sol` contract is paused due to lack of `whenNotPaused` modifier.

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L183-L197


e.g. `transferBackToLRTDepositPool()` function does the opposite operation and it has `whenNotPaused` modifier.

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L79

## Recommended Mitigation Steps
Consider to add `whenNotPaused` modifier to `transferAssetToNodeDelegator()` function.

```diff
    function transferAssetToNodeDelegator(
        uint256 ndcIndex,
        address asset,
        uint256 amount
    )
        external
        nonReentrant
++      whenNotPaused
        onlyLRTManager
        onlySupportedAsset(asset)
    {
        address nodeDelegator = nodeDelegatorQueue[ndcIndex];
        if (!IERC20(asset).transfer(nodeDelegator, amount)) {
            revert TokenTransferFailed();
        }
    }
```

# [L-03] `burnFrom` function burn tokens without an allowance.

## Description
`burnFrom` function can burn user tokens without an allowance. Despite this function can be called by trusted `BURNER_ROLE` this function can be unexpectedly called by mistake.

```solidity
function burnFrom(address account, uint256 amount) external onlyRole(BURNER_ROLE) whenNotPaused {
    _burn(account, amount);
}
```

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/RSETH.sol#L54-L56

## Recommended Mitigation Steps
Consider implement the next changes:

```diff
function burnFrom(address account, uint256 amount) external onlyRole(BURNER_ROLE) whenNotPaused {
++  _spendAllowance(account, msg.sender, amount);
    _burn(account, amount);
}
```

# [NC-01] `updateAssetStrategy()` doesn't used in constructor.

## Description
`updateAssetStrategy()` function used in `LRTConfig` to set `strategy` for `supportedAsset`.

When `LRTConfig.sol` contract is deployed it sets supported assets and their limits in the constructor, however it doesn't set strategy.

When `depositAssetIntoStrategy()` function is used it will revert since default strategy is equal to address(0) if admin forget to set strategy for asset. 

```solidity
IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
```

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L67

## Recommended Mitigation Steps
Consider to implement the following changes:

- use `updateAssetStrategy()` in the constructor to set strategy for supported assets that are added in the constructor as well.
- implement zero address check in `depositAssetIntoStrategy()` function to check that strategy isn't equal to address(0).


# [NC-02] Lack of check in `getAssetCurrentLimit()`.

## Description
In `getAssetCurrentLimit()` there is no check that `lrtConfig.depositLimitByAsset(asset) >= getTotalAssetDeposits(asset)`

```solidity
function getAssetCurrentLimit(address asset) public view override returns (uint256) { 
    return lrtConfig.depositLimitByAsset(asset) - getTotalAssetDeposits(asset);
}
```

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L56-L58


Due to this `depositAsset()` can revert with a reason unknown to the user.

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L132
## Recommended Mitigation Steps

Consider to add the following check:

```diff
function getAssetCurrentLimit(address asset) public view override returns (uint256) { 
++  if (getTotalAssetDeposits(asset) > lrtConfig.depositLimitByAsset(asset)) revert Error();
    return lrtConfig.depositLimitByAsset(asset) - getTotalAssetDeposits(asset);
}
```