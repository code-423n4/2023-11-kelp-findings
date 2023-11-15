QA (4 in total) 

1. GetAssetCurrentLimit
``` javascript
function getAssetCurrentLimit(address asset) public view override returns (uint256) {

return lrtConfig.depositLimitByAsset(asset) - getTotalAssetDeposits(asset);

}
```
This function can be viewed by everyone and returns the Currentlimit from an asset. 

``` javascript
if (depositAmount > getAssetCurrentLimit(asset)) {

revert MaximumDepositLimitReached();

}
```
This section of the `depositAsset` function checks whether the deposit amount exceeds the current limit obtained from `getAssetCurrentLimit(asset)`.  A malicious user could monitor the asset's limit using the public function and, as it approaches the limit, intentionally causes pending transactions from honest users to fail by depositing an amount that triggers the `revert ExceededMaximumDepositLimit()` condition. This could lead to a frustrating experience for users and eventually a loss of clients. Solution: You could change the `GetAssetCurrentLimit` function to `private` instead of public

2.  It is recommended to include the `pause` and `unpause` functions in the `LRTConfig.sol`. While the project incorporates the pause and unpause functions in most of the "scope.txt" contracts, the `LRTConfig.sol` contract currently lacks this feature. For safety and user reassurance, it would be prudent to enable pausing and unpausing for this contract as well.

Certainly! Here's a revised version with improved wording:

3. Consider adding `onlyLRTAdmin` to the `UpdatePriceOracleFor` function so that the admin has the capability to update the price oracle.

4. It's redundant to use both `onlyLRTAdmin` and `DEFAULT_ADMIN_ROLE` in the codebase while both terms have the same meaning. Choose one term for clarity and simplicity. For example, in the `updateAssetStrategy` function, you can replace `onlyRole(DEFAULT_ADMIN_ROLE)` with `onlyLRTAdmin` for consistency:

```javascript
function updateAssetStrategy(

address asset,

address strategy

)

external

onlyRole(DEFAULT_ADMIN_ROLE)

onlySupportedAsset(asset)

{

UtilLib.checkNonZeroAddress(strategy);

if (assetStrategy[asset] == strategy) {

revert ValueAlreadyInUse();

}

assetStrategy[asset] = strategy;
```

This change ensures that the terminology is consistent, making the codebase more readable and maintainable.