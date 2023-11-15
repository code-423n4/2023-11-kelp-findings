## Node delegators cannot be removed
There is a function for adding Node Delegators to the `nodeDelegatorQueue` array: `addNodeDelegatorContractToQueue()`. 
````
    function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {
        uint256 length = nodeDelegatorContracts.length;
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
````

The problem is that there is no possibility for removing them. I highly suggest adding this functionality for not having any further problems like a too large array that will end up making users and the protocol pay higher gas fees and maybe get transactions reverted.

## Max approve to EigenLayer its dangerous
For approving EigenLayer's strategies to take tokens from the NodeDelegators, admins will call the `maxApproveToEigenStrategyManager()` function.
````
    function maxApproveToEigenStrategyManager(address asset)
        external
        override
        onlySupportedAsset(asset)
        onlyLRTManager
    {
        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);
        IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);
    }
````
This will approve all the tokens that are and will be in the NodeDelegator contract to the EigenLayer's strategy. In my opinion this is a dangerous approach, since if EigenLayer gets hacked or compromised this will directly affect all the funds that the NodeDelegators hold also.

To mitigate this risk, I think that a safer approach could be approving only the amount that the contract is going to deposit into the strategy. This can be done by adding an approve to the `depositAssetIntoStrategy()` function.

````
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

      + // Add this line
      + IERC20(asset).approve(eigenlayerStrategyManagerAddress, balance);

        IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
    }
````