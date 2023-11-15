# GAS OPTIMIZATION

# SUMMARY
|      |  issue  |  instance  |
|------|---------|------------|
|[G‑01]|Avoid contract existence checks by using low level calls|5|
|[G‑02]|Optimize External Calls with Assembly for Memory Efficiency|2|
|[G‑03]|Cache external calls outside of loop to avoid re-calling function on each iteration|2|
|[G‑04]|Can Make The Variable Outside The Loop To Save Gas |3|
|[G‑05]|Expressions for constant values such as a call to¬†keccak256(), should use immutable rather than constant|2|
|[G‑06]|Amounts should be checked for 0 before calling a transfer|2|
|[G‑07]|Use constants instead of type(uintx).max|1|
|[G‑08]|Avoid emitting storage values|1|



## [G‑01] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

There are 5 instances of this issue:

save gas with this change : 5 * 100 = 500
totoal save gas = 500
```solidity
File:  src/LRTDepositPool.sol
83   assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L83

```solidity
File:  src/LRTOracle.sol
54   uint256 rsEthSupply = IRSETH(rsETHTokenAddress).totalSupply();

70   uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L54

```solidity
File:  src/NodeDelegator.sol
102     (IStrategy[] memory strategies,) =
              IEigenStrategyManager(eigenlayerStrategyManagerAddress).getDeposits(address(this));

123    return IStrategy(strategy).userUnderlyingView(address(this));              
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L102-L103

## [G-02] Optimize External Calls with Assembly for Memory Efficiency
Using interfaces to make external contract calls in Solidity is convenient but can be inefficient in terms of memory utilization. Each such call involves creating a new memory location to store the data being passed, thus incurring memory expansion costs.

Inline assembly allows for optimized memory usage by re-using already allocated memory spaces or using the scratch space for smaller datasets. This can result in notable gas savings, especially for contracts that make frequent external calls.

Additionally, using inline assembly enables important safety checks like verifying if the target address has code deployed to it using extcodesize(addr) before making the call, mitigating risks associated with contract interactions.


There are 5 instances of this issue:

```solidity
File:  src/LRTDepositPool.sol
156   IRSETH(rsethToken).mint(msg.sender, rsethAmountToMint);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L156


```diff
function _mintRsETH(address _asset, uint256 _amount) private returns (uint256 rsethAmountToMint) {
    (rsethAmountToMint) = getRsETHAmountToMint(_asset, _amount);

    address rsethToken = lrtConfig.rsETH();

-   // mint rseth for user
-   IRSETH(rsethToken).mint(msg.sender, rsethAmountToMint);
+   // Assembly implementation
+   assembly {
+       // Load the address of the rsethToken contract
+       let token := mload(0x40)
+       mstore(token, rsethToken)

+       // Load the signature of the mint function
+       let signature := mload(0x80)
+       mstore(signature, 0x40c10f19) // Function signature of "mint(address,uint256)"

+       // Load the function selector
+       let selector := and(0xffffffff00000000000000000000000000000000000000000000000000000000, keccak256(signature, 4))

+       // Prepare the data for the function call
+       let data := mload(0xC0)
+       mstore(data, selector) // Function selector
+       mstore(add(data, 0x04), caller) // Address of the caller (msg.sender)
+       mstore(add(data, 0x24), rsethAmountToMint) // rsethAmountToMint

+       // Call the mint function
+       let success := call(gas(), token, 0, data, 0x44, 0, 0)

+       // Check if the call was successful
+       if iszero(success) {
+           revert(0, 0)
+       }
+   }
}

```

```solidity
File:  src/NodeDelegator.sol
45   IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L45


## [G-03] Cache external calls outside of loop to avoid re-calling function on each iteration
Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

There are 2 instances of this issue:


```solidity
File:  src/LRTDepositPool.sol
83  assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L83


```diff
    {
+       IERC20 assetToken = IERC20(asset);
        // Question: is here the right place to have this? Could it be in LRTConfig?
-       assetLyingInDepositPool = IERC20(asset).balanceOf(address(this));
+       assetLyingInDepositPool = assetToken.balanceOf(address(this));

        uint256 ndcsCount = nodeDelegatorQueue.length;
        for (uint256 i; i < ndcsCount;) {
-           assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
+           assetLyingInNDCs += assetToken.balanceOf(nodeDelegatorQueue[i]);
            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
            unchecked {
                ++i;
            }
        }
    }
```



```solidity
File:  src/LRTOracle.sol
// @audit is ILRTDepositPool external
70   uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L70


## [G-04] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

There are 3 instances of this issue:

```solidity
File:   src/LRTOracle.sol
            address asset = supportedAssets[asset_idx];
            uint256 assetER = getAssetPrice(asset);

            uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L67-L70


```diff
function getRSETHPrice() external view returns (uint256 rsETHPrice) {
        address rsETHTokenAddress = lrtConfig.rsETH();
        uint256 rsEthSupply = IRSETH(rsETHTokenAddress).totalSupply();

        if (rsEthSupply == 0) {
            return 1 ether;
        }

        uint256 totalETHInPool;
        address lrtDepositPoolAddr = lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL);

        address[] memory supportedAssets = lrtConfig.getSupportedAssetList();
        uint256 supportedAssetCount = supportedAssets.length;
+       address asset
+       uint256 assetER
+       uint256 totalAssetAmt
        for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
-           address asset = supportedAssets[asset_idx];
+           asset = supportedAssets[asset_idx];
-           uint256 assetER = getAssetPrice(asset);
+           assetER = getAssetPrice(asset);
-           uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
+           totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
            totalETHInPool += totalAssetAmt * assetER;

            unchecked {
                ++asset_idx;
            }
        }

        return totalETHInPool / rsEthSupply;
    }

```

## [G-05] Expressions for constant values such as a call to¬†keccak256(), should use immutable rather than constant
The reason for this is that constant variables are evaluated at runtime and their value is included in the bytecode of the contract. This means that any expensive operations performed as part of the constant expression, such as a call to keccak256(), will be executed every time the contract is deployed, even if the result is always the same. This can result in higher gas costs.

```
bytes32 constant MY_HASH = keccak256("my string");
```
Alternatively, we could use an immutable variable, like so:
```
bytes32 immutable MY_HASH = keccak256("my string");
```

There are 2 instances of this issue:

```solidity
File:  src/RSETH.sol
21  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

22  bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L21-L22

```diff
-    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
+    bytes32 public immutable MINTER_ROLE = keccak256("MINTER_ROLE");
-    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
+    bytes32 public immutable BURNER_ROLE = keccak256("BURNER_ROLE");
```

## [G-06] Amounts should be checked for 0 before calling a transfer
Checking non-zero transfer values can avoid an expensive external call and save gas.

There are 2 instances of this issue:

```solidity
File:  src/LRTDepositPool.sol
194   if (!IERC20(asset).transfer(nodeDelegator, amount)) {
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L194


```diff
    {
// @audit add custom error in top  Amountgreaterzero()       
+       require(amount > 0) Amountgreaterzero();
        address nodeDelegator = nodeDelegatorQueue[ndcIndex];
        if (!IERC20(asset).transfer(nodeDelegator, amount)) {
            revert TokenTransferFailed();
        }
    }
```


```solidity
File:  src/NodeDelegator.sol
86  if (!IERC20(asset).transfer(lrtDepositPool, amount)) {
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L86


## [G-07] Use constants instead of type(uintx).max
Using constants instead of type(uintx).max can save gas in some cases because constants are precomputed at compile-time and can be reused throughout the code, whereas type(uintx).max requires computation at runtime

There are 1 instances of this issue:

```solidity
File:  src/NodeDelegator.sol
45   IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L45

```diff
contract NodeDelegator is INodeDelegator, LRTConfigRoleChecker, PausableUpgradeable, ReentrancyGuardUpgradeable {
+   // Define a constant for maximum uint256 value
+   uint256 constant MAX_UINT256 = 2**256 - 1; 
    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers();
    }

    /// @dev Initializes the contract
    /// @param lrtConfigAddr LRT config address
    function initialize(address lrtConfigAddr) external initializer {
        UtilLib.checkNonZeroAddress(lrtConfigAddr);
        __Pausable_init();
        __ReentrancyGuard_init();

        lrtConfig = ILRTConfig(lrtConfigAddr);
        emit UpdatedLRTConfig(lrtConfigAddr);
    }

    /// @notice Approves the maximum amount of an asset to the eigen strategy manager
    /// @dev only supported assets can be deposited and only called by the LRT manager
    /// @param asset the asset to deposit
    function maxApproveToEigenStrategyManager(address asset)
        external
        override
        onlySupportedAsset(asset)
        onlyLRTManager
    {
        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);
-       IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);
+       IERC20(asset).approve(eigenlayerStrategyManagerAddress, MAX_UINT256);
    }

```


## [G-08] Avoid emitting storage values
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

There are 1 instances of this issue:

```solidity
File:  src/LRTDepositPool.sol
204   emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L204