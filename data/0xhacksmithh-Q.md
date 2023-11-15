### [Low-01] No `minimum Amount(rsETH)` receive parameter absent in `depositAsset()`
Here we can see that User deposit asset via `depositAsset()` which take `asset address` and `asset depositAmount` as parameter
Then `rsethAmountMinted` calculated via `_mintRsETH()`
    - where it fetch corresponding asset price from oracle
    - then use formula (amount * assetPrice)/rsETHPrice to calculate `rsethAmountMinted`

Problem here is that if something go wrong with Oracle, then fetched price is mismatched with actual value
Then User may receive less amount than he intended, may be there a huge slippage.

```diff
    function depositAsset( 
        address asset,
        uint256 depositAmount,
+       uint256 minAmount
    )
        external
        whenNotPaused
        nonReentrant
        onlySupportedAsset(asset)
    {
        // checks
        if (depositAmount == 0) {
            revert InvalidAmount();
        }
        if (depositAmount > getAssetCurrentLimit(asset)) { 
            revert MaximumDepositLimitReached();
        }

        if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) { 
            revert TokenTransferFailed();
        }

        // interactions
-       uint256 rsethAmountMinted = _mintRsETH(asset, depositAmount); 
+       uint256 rsethAmountMinted = _mintRsETH(asset, depositAmount, minAmount); 

        emit AssetDeposit(asset, depositAmount, rsethAmountMinted);
    }
```

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L151-L157
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L119-L122
```

### Mitigation
To Overcome that `function` should also have a another parameter like `minAmountReceived` which inputed by user where User clearly says how much (slippage he can bear) `rsEth` he wanted to mint back.

And it should be check against `rsethAmountToMint` in `_mintRsETH()` before minting

```diff
-   function _mintRsETH(address _asset, uint256 _amount) private returns (uint256 rsethAmountToMint) {
+   function _mintRsETH(address _asset, uint256 _amount, uint _minAmount) private returns (uint256 rsethAmountToMint) {
        (rsethAmountToMint) = getRsETHAmountToMint(_asset, _amount);

+       require(rsethAmountToMint >= _minAmount, "error");
        address rsethToken = lrtConfig.rsETH();
        // mint rseth for user
        IRSETH(rsethToken).mint(msg.sender, rsethAmountToMint);
    }
```



### [Low-02] Attacker Can target a User for DOS in `depositAsset()`
According to `depositAsset()` the deposited Amount should not exceed asset Limit
```solidity
        if (depositAmount > getAssetCurrentLimit(asset)) {
            revert MaximumDepositLimitReached();
        }
```
### POC
- Let say current asset limit of `stETH` is 10_00
- Already `stETH` deposited is 990
- So now `getAssetCurrentLimit` = 10
- Now User-A go for a deposit of 10
- Attacker see that transaction, now he make a small amount deposit by front-running User-A
- Now `getAssetCurrentLimit` < 10
- User-A transaction failed due to check `if (depositAmount > getAssetCurrentLimit(asset))`

### Mitigation
Most classic way to deal with this issue is to just accept require amount of asset from the caller to reach the limit
For example see below POC
- let say limit of asset is 10
- 5 unit already deposited 
- User-A initiate Tx with 5 Unit
- Attacker front-run via depositing 1 unit
- Now As `newCurrentLimit < depositAmount` instead of `revert` `depositAsset()` only take 4(newCurrentLimit) unit from User-A

```diff
    function depositAsset( 
        address asset,
        uint256 depositAmount
    )
        external
        whenNotPaused
        nonReentrant
        onlySupportedAsset(asset)
    {
        // checks
        if (depositAmount == 0) {
            revert InvalidAmount();
        }
-       if (depositAmount > getAssetCurrentLimit(asset)) { 
-           revert MaximumDepositLimitReached();
-       }

+       if(depositedAmount > getAssetCurrentLimit(asset)){
+           depositedAmount = getAssetCurrentLimit(asset);
+       }    

        if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) { 
            revert TokenTransferFailed();
        }

        // interactions
       uint256 rsethAmountMinted = _mintRsETH(asset, depositAmount); 

        emit AssetDeposit(asset, depositAmount, rsethAmountMinted);
    }
```
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L119-L122
```

### [Low-03] There may be a possibility that `strategy` for an `asset` not present
Strategy for an asset is set via `updateAssetStrategy()` which set value in `assetStrategy[asset]` mapping
```solidity
function updateAssetStrategy( // @audit-issue may possible that there may be a no strategy for an asset
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
    }
```
There may be a possibility that for an asset, strategy is not set or set to zero address.

When deposit occurs to asset's Strategy in `depositAssetIntoStrategy()` it does not check that, strategy for that asset is actually present or not. In that case call will occur to zero address 
```solidity
    function depositAssetIntoStrategy(address asset)
        external
        override
        whenNotPaused
        nonReentrant
        onlySupportedAsset(asset)
        onlyLRTManager
    {
        address strategy = lrtConfig.assetStrategy(asset);
        IERC20 token = IERC20(asset);
        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

        uint256 balance = token.balanceOf(address(this));

        emit AssetDepositIntoStrategy(asset, strategy, balance);

        IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance); // @audit-ok external contract call
    }
```
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L51-L68
```

### [Low-04] Arrays only has `push` not `pop` method
NodeDelegator and supportedAsset arrays only have push method i.e only things added to array, what if there will some need to remove asset from supportAsset array.
No Pop or delisting method present.
```solidity
    function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin { 
        if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
            revert MaximumNodeDelegatorCountReached();
        }

        for (uint256 i; i < length;) {
            UtilLib.checkNonZeroAddress(nodeDelegatorContracts[i]);
            nodeDelegatorQueue.push(nodeDelegatorContracts[i]); 
            emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]);
            unchecked {
                ++i;
            }
        }
    }
```
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L169
```
```solidity
    function _addNewSupportedAsset(address asset, uint256 depositLimit) private {
        UtilLib.checkNonZeroAddress(asset);
        if (isSupportedAsset[asset]) {
            revert AssetAlreadySupported();
        }
        isSupportedAsset[asset] = true;
        supportedAssetList.push(asset); // @audit no remove option
        depositLimitByAsset[asset] = depositLimit;
        emit AddedNewSupportedAsset(asset, depositLimit);
    }
```
```
```

### [Low-05] Event missing in `updateAssetStrategy()` 
```solidity
    function updateAssetStrategy( // @audit-issue may possible that there may be a no strategy for an asset
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
    }
```
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L109-L122
```