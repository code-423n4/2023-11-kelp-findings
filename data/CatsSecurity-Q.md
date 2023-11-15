## [NC - 1] - Add `onlySupportedAsset(asset)` modifier to `getAssetBalance()` in NodeDelegator.sol
The modifier can be added to the function to catch the mistake earlier instead of reverting letter
```diff
  function getAssetBalance(address asset) 
external 
view 
override 
+ onlySupportedAsset(asset)
returns (uint256) {
        address strategy = lrtConfig.assetStrategy(asset);
        return IStrategy(strategy).userUnderlyingView(address(this));
    }
   ```
   
   https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L121-L124
   
  ## [NC-2] `updateMaxNodeDelegatorCount` can increase or decrease the maxDelegatorCount but there is no mechanism to remove the extra delegators in case the maxNodeDelegatorCount is decreased from current value
  
  The following function can either increase or decrease the max number of node delegators, lets say currently there are 10 node delegators, all handling the funds. There is no mechanism to remove extras from `nodeDelegatorQueue` in case the max is decreased from current value.
  
  ```solidity
      function updateMaxNodeDelegatorCount(uint256 maxNodeDelegatorCount_) external onlyLRTAdmin {
        maxNodeDelegatorCount = maxNodeDelegatorCount_;
        emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount);
    }
```
    
## [NC-3] Emit the depositers address too in `depositAsset()`
    This instance was not reported in the bot so adding it here.
`depositAsset` function should emit the address of the depositer too along with the other parameters for easy off-chain tracking.

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
        if (depositAmount > getAssetCurrentLimit(asset)) {
            revert MaximumDepositLimitReached();
        }

        // if it returns true anyways, what will happen?  it will not revert and transfer will happen
        if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) {
            revert TokenTransferFailed();
        }

        // interactions
        uint256 rsethAmountMinted = _mintRsETH(asset, depositAmount);

        emit AssetDeposit(asset, 
                                  depositAmount, 
                                  rsethAmountMinted,
+                               msg.sender);
    }
```
## [NC-4] There is no mechanism to handle in case the deposit limit is decreased below the current deposits in the contracts.
Deposit limit can be increased or decreased anytime but the contracts implement no mechanism to handle in case it is decreased and current deposits in the system are greater than the deposit limit. There should be a mechanism whenever the limit is decreased below current funds the extra should be immediately taken out of eigen layer

```solidity 
    function updateAssetDepositLimit(
        address asset,
        uint256 depositLimit
    )
        external
        onlyRole(LRTConstants.MANAGER)
        onlySupportedAsset(asset)
    {
        depositLimitByAsset[asset] = depositLimit;
        emit AssetDepositLimitUpdate(asset, depositLimit);
    }
 ```

## [NC-5] Opezeppelin enumerable set can be used in `LRTDepositPool.so` to better handle the node delegator queue using its build in functions
Enumerable sets have a lot of built-in functions to easily add and remove items from such structures and are also tested and secure. Use them to add and remove items from the nodeDelegator queue, will be more readable too.

```
    address[] public nodeDelegatorQueue;
```

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L22


## [NC-6] Use Address instead of Addr for more readability

Replace `lrtConfigAddr` with `lrtConfigAddress`

```solidity    function initialize(address lrtConfigAddr) external initializer {
        UtilLib.checkNonZeroAddress(lrtConfigAddr);
        __Pausable_init();
        __ReentrancyGuard_init();
        maxNodeDelegatorCount = 10;
        lrtConfig = ILRTConfig(lrtConfigAddr);
        emit UpdatedLRTConfig(lrtConfigAddr);
    }
```

## [NC-6] `getLstToken`  is unnecessary, being not used anywhere, if it is for frontend simply use the `supportedAssetList` to get the desired token

```diff
-    function getLSTToken(bytes32 tokenKey) external view override returns (address) {
-       return tokenMap[tokenKey];
-    }
```

## [NC-7] Keep track of the token each user deposited and the amount deposited by an address, may help when implementing the withdraw flow.
In the deposit function we can keep track of each token an address deposited and the amount deposited, while implementing the withdraw there can be multiple nuances and such structure may help in doing right calculations.

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
        if (depositAmount > getAssetCurrentLimit(asset)) {
            revert MaximumDepositLimitReached();
        }

        // if it returns true anyways, what will happen?  it will not revert and transfer will happen
        if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) {
            revert TokenTransferFailed();
        }

        // interactions
        uint256 rsethAmountMinted = _mintRsETH(asset, depositAmount);

+     amountDeposited[msg.sender][asset] = depositAmount;

        emit AssetDeposit(asset, depositAmount, rsethAmountMinted);
    }
```

