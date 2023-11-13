https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/RSETH.sol#L14-L20

1- contract does inherit it's own interface 

`RSETH.SOL` is not inheriting `IRSETH.SOL`

Contract implementations should inherit their interfaces. Extending an interface ensures that all function signatures are correct, and catches mistakes introduced by typos 

```
contract RSETH is
    Initializable,
    LRTConfigRoleChecker,
    ERC20Upgradeable,
    PausableUpgradeable,
    AccessControlUpgradeable
```
##FIX 

```
contract RSETH is IRSETH
    Initializable,
    LRTConfigRoleChecker,
    ERC20Upgradeable,
    PausableUpgradeable,
    AccessControlUpgradeable
```

2- Upgradable Tokens can break the contract behavior.


Some tokens (e.g. USDC, USDT) are upgradable, allowing the token owners to make arbitrary modifications to the logic of the token at any point in time.

A change to the token semantics can break any smart contract that depends on the past behaviour.
Consider introducing logic that will freeze interactions with the token in question if an upgrade is detected.

For example TUSD adapter used by MakerDao ( https://github.com/makerdao/dss-deploy/blob/7394f6555daf5747686a1b29b2f46c6b2c64b061/src/join.sol#L321)

3- https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L80-L81

New supported asset contract address might not be ERC20-Compatible.

it is good practice to validate whether a new asset address is an ERC20-compatible contract before adding it as a supported asset. This validation helps ensure that only valid `ERC20` tokens are added to the supported asset list.

To check if an address corresponds to an `ERC20` contract, you can utilize the `ERC20` interface's ` totalSupply() ` function. If the  `totalSupply()`  function exists and returns a value without throwing an exception, it indicates that the address is an `ERC20` contract. Here's an example of how you can perform the check:

 ```
function isERC20(address asset) private view returns (bool) {
    uint256 constant DUMMY_VALUE = 0;
    try IERC20(asset).totalSupply() returns (uint256) {
        return true;
    } catch {
        return false;
    }
}
```

In the  `_addNewSupportedAsset ` function, you can add a check for |`ERC20` compatibility before adding the asset to the supported asset list. Here's an updated version of the function with the `ERC20`check included:

``` 
function _addNewSupportedAsset(address asset, uint256 depositLimit) private {
    UtilLib.checkNonZeroAddress(asset);
    if (isSupportedAsset[asset]) {
        revert AssetAlreadySupported();
    }
    if (!isERC20(asset)) {
        revert AssetNotERC20();
    }
    isSupportedAsset[asset] = true;
    supportedAssetList.push(asset);
    depositLimitByAsset[asset] = depositLimit;
    emit AddedNewSupportedAsset(asset, depositLimit);
}
 
```
By including the  `isERC20`  check, you can ensure that only ERC20-compatible contracts are added as supported assets.

4- https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L80-L81

Asset address can be added but not removed


```
    function _addNewSupportedAsset(address asset, uint256 depositLimit) private {
        UtilLib.checkNonZeroAddress(asset);
        if (isSupportedAsset[asset]) {
            revert AssetAlreadySupported();
        }
        isSupportedAsset[asset] = true;
        supportedAssetList.push(asset);
        depositLimitByAsset[asset] = depositLimit;
        emit AddedNewSupportedAsset(asset, depositLimit);
    }
```

Where there is a push there should be a pop, if an asset becomes redundant or the Dao chooses not to use such asset anymore it can't be removed as there's no remove `asset`functionality.

##FIX 
Include a `removeAsset` function.