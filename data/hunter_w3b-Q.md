## [L-01] Deposit limits can be updated by the manager role, but there is no check to prevent reducing limits. This could lock user assets

The addNewSupportedAsset function allows the manager role to update the deposit limit amount for any supported asset. However, there is no check to prevent reducing this limit to a value below what users have already deposited.

```solidity
File: src/LRTConfig.sol

73    function addNewSupportedAsset(address asset, uint256 depositLimit) external onlyRole(LRTConstants.MANAGER) {
74        _addNewSupportedAsset(asset, depositLimit);
75    }
76
77    /// @dev private function to add a new supported asset
78    /// @param asset Asset address
79    /// @param depositLimit Deposit limit for the asset
80    function _addNewSupportedAsset(address asset, uint256 depositLimit) private {
81        UtilLib.checkNonZeroAddress(asset);
82        if (isSupportedAsset[asset]) {
83            revert AssetAlreadySupported();
84        }
85        isSupportedAsset[asset] = true;
86        supportedAssetList.push(asset);
87        depositLimitByAsset[asset] = depositLimit;
88        emit AddedNewSupportedAsset(asset, depositLimit);
89    }
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L73-L89

This could potentially lock funds if a limit is reduced after a user has deposited more than the new lower limit. An attacker with manager role privileges could maliciously lock funds in this way. The deposit limit should always be increased or maintained, never decreased, to avoid locking user funds.

## [L-02] Strategies can be updated by any admin, but there is no logic to handle existing funds. Strategy changes could lose user asset

The updateAssetStrategy function allows any admin to change the strategy address for how an asset is managed. However, there is no logic defined in the contract for how existing deposited funds should be handled during a strategy change. Without proper handling, funds could be lost or inaccessible if sent to the new strategy before it is ready. An attacker with admin privileges could maliciously change the strategy and cause user funds to be lost or inaccessible. Before allowing strategy changes, the contract should include logic to withdraw existing funds from the old strategy and ensure they are securely managed during the transition.

```solidity
File: src/LRTConfig.sol

109    function updateAssetStrategy(
110        address asset,
111        address strategy
112    )
113        external
114        onlyRole(DEFAULT_ADMIN_ROLE)
115        onlySupportedAsset(asset)
116    {
117        UtilLib.checkNonZeroAddress(strategy);
118       if (assetStrategy[asset] == strategy) {
119            revert ValueAlreadyInUse();
120        }
121        assetStrategy[asset] = strategy;
122    }
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L109-L122

## [L-03] Strategies can be set per asset, but contract may expect global strategy behavior mismatch

The architecture of setting strategies per individual asset through the assetStrategy mapping may cause a misalignment with how the contract is intended to handle strategies.

```solidity
File: src/LRTConfig.sol

17    mapping(address token => address strategy) public override assetStrategy;

```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L17

If the code expects there to only be a single global strategy for all assets, the per-asset setting could lead to unexpected behavior.

For example, asset handling logic may send all assets to a single strategy address instead of the one configured for that specific asset. This mismatched design between the strategy setting and underlying logic could result in assets being sent to incorrect strategies and assets being mishandled or lost as a result.

## [L-04] Total balance calculation iterates queue but doesn't consider new deposits in loop

The getTotalAssetDeposits function calculates the total asset balance across the deposit pool and node delegate contracts. It does this by iterating the node delegate queue and summing the balances of each contract. However, this calculation does not properly consider the possibility of new deposits being made to the deposit pool contract during the iteration of the queue. Since external calls are asynchronous, it is possible for a deposit to be made after the deposit pool balance is read, but before balances of node delegates are checked. This would result in the total balance being reported incorrectly low as it would not account for the new deposit amount. The balance calculation needs to prevent re-entrancy or take a snapshot of relevant balances upfront to avoid this potential inaccuracy.

```solidity
File: src/LRTDepositPool.sol

47    function getTotalAssetDeposits(address asset) public view override returns (uint256 totalAssetDeposit) {
48        (uint256 assetLyingInDepositPool, uint256 assetLyingInNDCs, uint256 assetStakedInEigenLayer) =
49            getAssetDistributionData(asset);
50        return (assetLyingInDepositPool + assetLyingInNDCs + assetStakedInEigenLayer);
51    }



71    function getAssetDistributionData(address asset)
72        public
73        view
74        override
75        onlySupportedAsset(asset)
76        returns (uint256 assetLyingInDepositPool, uint256 assetLyingInNDCs, uint256 assetStakedInEigenLayer)
77    {
78        // Question: is here the right place to have this? Could it be in LRTConfig?
79        assetLyingInDepositPool = IERC20(asset).balanceOf(address(this));
80
81        uint256 ndcsCount = nodeDelegatorQueue.length;
82        for (uint256 i; i < ndcsCount;) {
83            assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
84            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
85            unchecked {
86                ++i;
87            }
88        }
89    }
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L71

## [L-05] Unused Asset Strategy Logic

The asset strategy concept is defined but not properly implemented or leveraged. By not passing the asset to strategy contracts, important logic like customized minting calculations cannot be executed. This undermines the extensibility of the protocol and may lead to unexpected behaviors.

The `_mintRsETH`() call does not pass the asset to allow custom strategy handling.

```solidity
File: src/LRTDepositPool.sol

    function _mintRsETH(address _asset, uint256 _amount) private returns (uint256 rsethAmountToMint) {
        (rsethAmountToMint) = getRsETHAmountToMint(_asset, _amount);

        address rsethToken = lrtConfig.rsETH();
        // mint rseth for user
        IRSETH(rsethToken).mint(msg.sender, rsethAmountToMint);
    }
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L151-L157

## [L-06] Insufficient Allowance Check

By not checking the allowance before calling transferFrom, token transfers could fail or revert if the allowance is not high enough. This could lead to asset loss and undermine the integrity of deposit/withdrawal functionality.

The depositAsset function does not check the allowance before calling transferFrom.

```solidity
File: src/LRTDepositPool.sol

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

        if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) {
            revert TokenTransferFailed();
        }

        // interactions
        uint256 rsethAmountMinted = _mintRsETH(asset, depositAmount);

        emit AssetDeposit(asset, depositAmount, rsethAmountMinted);
    }
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L119-L144

- Check allowance before transferFrom using allowance(owner, contract)
- Revert if allowance < amount to transfer

## [L-07] Strategy functions are called without validating success or expected return values

By not checking the return values of calls to strategy functions like depositIntoStrategy(), failures or unexpected return values could be ignored. This can lead to funds being deposited into invalid states without detections.

Function calls in depositAssetIntoStrategy() do not validate the call success or expected return value:

```solidity
File: src/NodeDelegator.sol

67        IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L67

## [L08] Lack of Limits on Minting and Burning Allows Uncontrolled Token Inflation/Deflation

The mint and burn functions in the RSETH token contract have no limitations on the amounts of tokens that can be created or destroyed.

```solidity
File: src/RSETH.sol

    function mint(address to, uint256 amount) external onlyRole(MINTER_ROLE) whenNotPaused {
        _mint(to, amount);
    }

        function burnFrom(address account, uint256 amount) external onlyRole(BURNER_ROLE) whenNotPaused {
        _burn(account, amount);
    }
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L47-L57

An attacker who gains the minter or burner role could mint or burn arbitrary quantities of tokens, causing rampant inflation or deflation of the total supply. Without caps on mint/burn amounts, a bad actor could manipulate the supply however they choose without constraint. This could severely compromise the stability and value of the RSETH token. Hard or configurable limits should be imposed to prevent such exploitation and maintain control over the token's emission schedule and long term monetary policy.
