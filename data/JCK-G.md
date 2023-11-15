
## Gas Optimizations

| Number | Issue | Instances |
|--------|-------|-----------|
|[G-01]| Use a do while loop instead of a for loop    |  4  | |
|[G-02]| Don’t cache calls that are only used once    |  8  | |
|[G-03]| Possible Optimizations in LRTConfig.sol    |  4  | |
|[G-04]| Do not cache immutable values    |  1  | |
|[G-05]| Counting down in for statements is more gas efficient    |  4  | |
|[G-06]| Avoid Unnecessary Public Variables    |  5  | |
|[G-07]| Cache external calls outside of loop to avoid re-calling function on each iteration    |  3  | |
|[G-08]| Use hardcoded address instead of address(this)    |  6  | |
|[G-09]| Use assembly for loops    |  3  | |
|[G-10]| Declare the variables outside the loop    |  3  | | 
|[G-11]| Immutable over constant  |  9  | |

## [G-01] Use a do while loop instead of a for loop

A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity
file: blob/main/src/NodeDelegator.sol

109   for (uint256 i = 0; i < strategiesLength;) {

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L109

### foundry test only used for NodeDelegator.sol

Before gas value:
[PASS] test_GetAssetBalances() (gas: 277672)

After gas vlue;
[PASS] test_GetAssetBalances() (gas: 277617)

```solidity
file: blob/main/src/LRTDepositPool.sol

82   for (uint256 i; i < ndcsCount;) {

168  for (uint256 i; i < length;) {

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L82


```solidity
file: blob/main/src/LRTOracle.sol

66  for (uint16 asset_idx; asset_idx < supportedAssetCount;) {

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L66


## [G-02] Don’t cache calls that are only used once


```solidity
file: blob/main/src/NodeDelegator.sol

100   address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L100
       
Before gas value:
[PASS] test_GetAssetBalances() (gas: 277672)

After gas value:
[PASS] test_GetAssetBalances() (gas: 277594)


```solidity
file: blob/main/src/NodeDelegator.sol

122   address strategy = lrtConfig.assetStrategy(asset);

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L122

Before gas value:
[PASS] test_GetAssetBalance() (gas: 164106)

After gas value:
[PASS] test_GetAssetBalance() (gas: 164095)

```solidity
file: blob/main/src/LRTDepositPool.sol

105   address lrtOracleAddress = lrtConfig.getContract(LRTConstants.LRT_ORACLE);

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L105

Before gas value:
[PASS] test_GetRsETHAmountToMint() (gas: 53143)

After gas vlue:
[PASS] test_GetRsETHAmountToMint() (gas: 53131)


```solidity
file: blob/main/src/LRTDepositPool.sol

141   uint256 rsethAmountMinted = _mintRsETH(asset, depositAmount);

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L141

Before gas value:
[PASS] test_DepositAsset() (gas: 174587)

After gas value:
[PASS] test_DepositAsset() (gas: 174566)

```solidity
file: blob/main/src/LRTDepositPool.sol

154  address rsethToken = lrtConfig.rsETH();

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L154

Before Gas value:
[PASS] test_GetRsETHAmountToMint() (gas: 53143)

After gas value:
[PASS] test_GetRsETHAmountToMint() (gas: 53131)


```solidity
file: blob/main/src/LRTOracle.sol

53   address rsETHTokenAddress = lrtConfig.rsETH();

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#l53

Before Gas value:
[PASS] test_FetchRSETHPrice() (gas: 76252)

After gas value:
[PASS] test_FetchRSETHPrice() (gas: 76240)


```solidity
file: blob/main/src/NodeDelegator.sol

44  address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L44

Before Gas value:
[PASS] test_MaxApproveToEigenStrategyManager() (gas: 73110)

After gas value:
[PASS] test_MaxApproveToEigenStrategyManager() (gas: 73109)


```solidity
file: blob/main/src/NodeDelegator.sol

61    address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L61

Before Gas value:
[PASS] test_DepositAssetIntoStrategy() (gas: 161218)

After gas value:
[PASS] test_DepositAssetIntoStrategy() (gas: 161211)


## [G-03] Possible Optimizations in LRTConfig.sol

General Optimization
Reducing the number of state variable updates can save gas. In Ethereum, every storage operation costs a significant amount of gas. Therefore, optimizing the contract to minimize storage operations can lead to substantial gas savings.

### Possible Optimization in function _addNewSupportedAsset()

In the _addNewSupportedAsset functions, the isSupportedAsset mapping is updated twice. This can be optimized to a single update, which would save gas.

```solidity
file: blob/main/src/LRTConfig.sol

80   function _addNewSupportedAsset(address asset, uint256 depositLimit) private {
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
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L80-L89

### foundry test only for _addNewSupportedAsset()

Before gas value:

[PASS] test_AddNewSupportedAsset() (gas: 793992)

After gas value:
[PASS] test_AddNewSupportedAsset() (gas: 773815)

### Possible Optimization

In the _setToken, _setContract and functions, the tokenMap, contractMap and assetStrategy mapping is updated twice. This can be optimized to a single update, which would save gas.

```solidity
file: blob/main/src/LRTConfig.sol

156    function _setToken(bytes32 key, address val) private {
        UtilLib.checkNonZeroAddress(val);
        if (tokenMap[key] == val) {
            revert ValueAlreadyInUse();
        }
        tokenMap[key] = val;
        emit SetToken(key, val);
    }

172  function _setContract(bytes32 key, address val) private {
        UtilLib.checkNonZeroAddress(val);
        if (contractMap[key] == val) {
            revert ValueAlreadyInUse();
        }
        contractMap[key] = val;
        emit SetContract(key, val);
    }

109  function updateAssetStrategy(
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
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L156-L163




## [G-04] Do not cache immutable values

```solidity
file: blob/main/src/LRTDepositPool.sol

106  ILRTOracle lrtOracle = ILRTOracle(lrtOracleAddress);

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L106

before gas vlaue:

[PASS] test_GetRsETHAmountToMint() (gas: 53143)

After gas value:
[PASS] test_GetRsETHAmountToMint() (gas: 53131)



## [G‑05] Counting down in for statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

```solidity
file: blob/main/src/LRTDepositPool.sol

82   for (uint256 i; i < ndcsCount;) {

168  for (uint256 i; i < length;) {

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L82


```solidity
file: blob/main/src/LRTOracle.sol

66  for (uint16 asset_idx; asset_idx < supportedAssetCount;) {

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L66

```solidity
file: blob/main/src/NodeDelegator.sol

109   for (uint256 i = 0; i < strategiesLength;) {

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L109



### Test Code

```solidity
contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;
    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }
    function testGas() public {
        c0.AddNum();
        c1.AddNum();
    }
}
contract Contract0 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=0;i<=9;i++){
            _num = _num +1;
        }
        num = _num;
    }
}
contract Contract1 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=9;i>=0;i--){
            _num = _num +1;
        }
        num = _num;
    }
}
```

## [G-06] Avoid Unnecessary Public Variables

Public storage variables increase the contract's size due to the implicit generation of public getter functions. This makes the contract larger and could increase deployment and interaction costs.

If you do not require other contracts to read these variables, consider making them private or internal.


```solidity
file: blob/main/src/LRTDepositPool.sol

20   uint256 public maxNodeDelegatorCount;

22   address[] public nodeDelegatorQueue;

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L20

```solidity
file: blob/main/src/utils/LRTConfigRoleChecker.sol

14   ILRTConfig public lrtConfig;

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConfigRoleChecker.sol#L14

```solidity
file: blob/main/src/LRTConfig.sol

19  address[] public supportedAssetList;

21  address public rsETH;

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L19


## [G-07] Cache external calls outside of loop to avoid re-calling function on each iteration

Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

```solidity
file: blob/main/src/LRTDepositPool.sol

84    assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L84

```solidity
file: blob/main/src/LRTOracle.sol 

70    uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L70

```solidity
file: blob/main/src/NodeDelegator.sol
 
111    assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L111


## [G-08] Use hardcoded address instead of address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this. Refrences

```solidity
file:  blob/main/src/LRTDepositPool.sol

79     assetLyingInDepositPool = IERC20(asset).balanceOf(address(this));

136    if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) {

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L79


```solidity
file: blob/main/src/NodeDelegator.sol

63   uint256 balance = token.balanceOf(address(this));

103  IEigenStrategyManager(eigenlayerStrategyManagerAddress).getDeposits(address(this));

111   assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));

123   return IStrategy(strategy).userUnderlyingView(address(this));

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L63

## [G-09] Use assembly for loops

In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are scratch space (0x00-0x20), the free memory pointer (0x40), and the zero slot (0x60). This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

Note that in order to do this optimization safely we will need to cache and restore the free memory pointer after the loop. We will also set the zero slot (0x60) back to 0.

```solidity
file: blob/main/src/LRTDepositPool.sol

82   for (uint256 i; i < ndcsCount;) {
            assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
            unchecked {
                ++i;
            }
        }

168   for (uint256 i; i < length;) {
            UtilLib.checkNonZeroAddress(nodeDelegatorContracts[i]);
            nodeDelegatorQueue.push(nodeDelegatorContracts[i]);
            emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]);
            unchecked {
                ++i;
            }
        }


```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L82-L87


```solidity
file: blob/main/src/NodeDelegator.sol

109  for (uint256 i = 0; i < strategiesLength;) {
            assets[i] = address(IStrategy(strategies[i]).underlyingToken());
            assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
            unchecked {
                ++i;
            }
        }

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L109-L115


## [G-10] Declare the variables outside the loop

```solidity
file:  blob/main/src/LRTOracle.sol

67     address asset = supportedAssets[asset_idx];

68     uint256 assetER = getAssetPrice(asset);

69     uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L67

Before Gas value:
[PASS] test_FetchRSETHPrice() (gas: 76252)

After Gas value:
[PASS] test_FetchRSETHPrice() (gas: 76207)


## [G-11] Immutable over constant

The use of constant keccak variables results in extra hashing whenever the variable is used, increasing gas costs relative to just storing the output hash. Changing to immutable will only perform hashing on contract deployment which will save gas.

```solidity
file: blob/main/src/RSETH.sol

21  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

22   bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#l21


```solidity
file: blob/main/src/utils/LRTConstants.sol

7      bytes32 public constant R_ETH_TOKEN = keccak256("R_ETH_TOKEN");
    //stETH token
    bytes32 public constant ST_ETH_TOKEN = keccak256("ST_ETH_TOKEN");
    //cbETH token
    bytes32 public constant CB_ETH_TOKEN = keccak256("CB_ETH_TOKEN");

    //contracts
    bytes32 public constant LRT_ORACLE = keccak256("LRT_ORACLE");
    bytes32 public constant LRT_DEPOSIT_POOL = keccak256("LRT_DEPOSIT_POOL");
    bytes32 public constant EIGEN_STRATEGY_MANAGER = keccak256("EIGEN_STRATEGY_MANAGER");

    //Roles
    bytes32 public constant MANAGER = keccak256("MANAGER");

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConstants.sol#L7-L19