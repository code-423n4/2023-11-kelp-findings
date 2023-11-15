## Gas optimizations

## Sumary

| No|Issue|Instances|
|-|:-|:-:|
|[G-01]|Change public function visibility to external|4|
|[G-02]| Use calldata instead of memory for function parameters|3|
|[G-03]|Use hardcode address instead address(this)|6|
|[G-04]|Modifiers are redundant if used only once or not used at all.|1|
|[G-05]|Use function instead of modifiers |2|
|[G-06]|Not using the named return variable when a function returns, wastes deployment gas|1|
|[G-07]| Use constants instead of type(uintx).max|1|
|[G-08]|Non efficient zero initialization|1|
|[G-09]|Should use arguments instead of state variable |17|
|[G-10]|State Variable can be packed into fewer storage slots|4|
|[G-11]|Expressions for constant values such as a call to keccak256()|2|
|[G-12]|Pre-increments and pre-decrements are cheaper than post-increments and post-decrements|4|
|[G-13]|Use uint256(1)/uint256(2) instead for true and false boolean states|1|
|[G-14]|Loop best practice to save gas|4|
|[G-15]|+= costs more gas than = + for state variables|2|
|[G-16]|Using calldata instead of memory for read-only arguments in external functions saves gas|2|
|[G-17]|Use assembly to emit events|17|


## [G-01] Change public function visibility to external
Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.

```solidity

file: LRTDepositPool.sol

47  function getTotalAssetDeposits(address asset) public view override returns (uint256 totalAssetDeposit) 

56   function getAssetCurrentLimit(address asset) public view override returns (uint256) 

71     function getAssetDistributionData(address asset)
        public
        view
        override
        onlySupportedAsset(asset)
        returns (uint256 assetLyingInDepositPool, uint256 assetLyingInNDCs, uint256 assetStakedInEigenLayer)

95   function getRsETHAmountToMint(
        address asset,
        uint256 amount
    )
        public
        view
        override
      returns (uint256 rsethAmountToMint)        
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L47
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L56
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L71
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L95

```solidity

file: LRTOracle.sol

45   function getAssetPrice(address asset) public view onlySupportedAsset(asset) returns (uint256) {
        return IPriceFetcher(assetPriceOracle[asset]).getAssetPrice(asset);
    }
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L45

## [G-02] Use calldata instead of memory for function parameters
If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

```solidity

file: LRTConfig.sol#

135     function getSupportedAssetList() external view override returns (address[] memory) {
        return supportedAssetList;
    }

62   function getNodeDelegatorQueue() external view override returns (address[] memory) {
        return nodeDelegatorQueue;
    }

94   function getAssetBalances()
        external
        view
        override
        returns (address[] memory assets, uint256[] memory assetBalances)
    {
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L135
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L62
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L94

### [G-03] Use hardcode address instead address(this)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address.

```solidity

file: LRTDepositPool.sol

79   assetLyingInDepositPool = IERC20(asset).balanceOf(address(this));

136    if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) {
            revert TokenTransferFailed();
        }
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L79
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L136

```solidity

file: NodeDelegator.sol

63  uint256 balance = token.balanceOf(address(this));

103   IEigenStrategyManager(eigenlayerStrategyManagerAddress).getDeposits(address(this));

111   assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));

123   return IStrategy(strategy).userUnderlyingView(address(this));
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L63
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L103
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L111
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L123

## [G-04] Modifiers are redundant if used only once or not used at all. 

```solidity

file: utils/LRTConfigRoleChecker.sol

20   modifier onlyLRTManager() {
        if (!IAccessControl(address(lrtConfig)).hasRole(LRTConstants.MANAGER, msg.sender)) {
            revert ILRTConfig.CallerNotLRTConfigManager();
        }
        _;
    }

    modifier onlyLRTAdmin() {
        bytes32 DEFAULT_ADMIN_ROLE = 0x00;
        if (!IAccessControl(address(lrtConfig)).hasRole(DEFAULT_ADMIN_ROLE, msg.sender)) {
            revert ILRTConfig.CallerNotLRTConfigAdmin();
        }
        _;
    }

    modifier onlySupportedAsset(address asset) {
        if (!lrtConfig.isSupportedAsset(asset)) {
            revert ILRTConfig.AssetNotSupported();
        }
        _;
40     }
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConfigRoleChecker.sol#L20-L40

## [G-05] Use function instead of modifiers 

```solidity

file: LRTConfig.sol

28   modifier onlySupportedAsset(address asset) 


```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L28

```solidity

file: utils/LRTConfigRoleChecker.so

20 20   modifier onlyLRTManager() {
        if (!IAccessControl(address(lrtConfig)).hasRole(LRTConstants.MANAGER, msg.sender)) {
            revert ILRTConfig.CallerNotLRTConfigManager();
        }
        _;
    }

    modifier onlyLRTAdmin() {
        bytes32 DEFAULT_ADMIN_ROLE = 0x00;
        if (!IAccessControl(address(lrtConfig)).hasRole(DEFAULT_ADMIN_ROLE, msg.sender)) {
            revert ILRTConfig.CallerNotLRTConfigAdmin();
        }
        _;
    }

    modifier onlySupportedAsset(address asset) {
        if (!lrtConfig.isSupportedAsset(asset)) {
            revert ILRTConfig.AssetNotSupported();
        }
        _;
40    }
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConfigRoleChecker.sol#L20-L40

## [G-06]  Not using the named return variable when a function returns, wastes deployment gas

```solidity

file: LRTConfig.sol

136  return supportedAssetList;
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L136

## [G-07] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity

file: NodeDelegator.sol

45   IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L45

## [G-08] Non efficient zero initialization
 
Solidity does not recognize null as a value, so uint variables are initialized to zero. Setting a uint variable to zero is redundant and can waste gas.

```solidity

file: NodeDelegator.sol

109 for (uint256 i = 0; i < strategiesLength;) 
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L109

## [G-09] Should use arguments instead of state variable 

state variables should not used in emit  ,  This will save near 97 gas   

```solidity

file: LRTConfig.sol

88      emit AddedNewSupportedAsset(asset, depositLimit);

103  emit AssetDepositLimitUpdate(asset, depositLimit);

162   emit SetToken(key, val);

178  emit SetContract(key, val);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L88
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L103
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L162
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L178

```solidity

file: LRTDepositPool.sol

37   emit UpdatedLRTConfig(lrtConfigAddr);

143  emit AssetDeposit(asset, depositAmount, rsethAmountMinted);

171  emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]);

204  emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L37
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L143
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L171
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L204

```solidity

file:NodeDelegator.sol

32  emit UpdatedLRTConfig(lrtConfigAddr);

65 emit AssetDepositIntoStrategy(asset, strategy, balance);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L32
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L65

```solidity

file:LRTOracle.sol

34  emit UpdatedLRTConfig(lrtConfigAddr);

98    emit AssetPriceOracleUpdate(asset, priceOracle);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L34
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L98

```solidity

file: RSETH.sol

41   emit UpdatedLRTConfig(lrtConfigAddr);

76         emit UpdatedLRTConfig(_lrtConfig);

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L41
https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L76

```solidity

file: oracles/ChainlinkPriceOracle.sol

31  emit UpdatedLRTConfig(lrtConfig_);

48   emit AssetPriceFeedUpdate(asset, priceFeed);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L31
https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L48

```solidity

file: utils/LRTConfigRoleChecker.sol

50   emit UpdatedLRTConfig(lrtConfigAddr);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConfigRoleChecker.sol#L50

## [G-10] State Variable can be packed into fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas)

```solidity

file: LRTConfig.sol

13     mapping(bytes32 tokenKey => address tokenAddress) public tokenMap;
    mapping(bytes32 contractKey => address contractAddress) public contractMap;
    mapping(address token => bool isSupported) public isSupportedAsset;
    mapping(address token => uint256 amount) public depositLimitByAsset;
17    mapping(address token => address strategy) public override assetStrategy;

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L13-L17

```solidity

file: LRTDepositPool.sol

20   uint256 public maxNodeDelegatorCount;

22    address[] public nodeDelegatorQueue;

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L20-L22

```solidity

file: LRTOracle.sol

20   mapping(address asset => address priceOracle) public override assetPriceOracle;
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L20

```solidity

file: oracles/ChainlinkPriceOracle.sol

18  mapping(address asset => address priceFeed) public override assetPriceFeed;

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L18

## [G-11] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

```solidity

file: RSETH.sol

21  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
22  bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");


```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L21-L22

```solidity

file: utils/LRTConstants.sol

7   bytes32 public constant R_ETH_TOKEN = keccak256("R_ETH_TOKEN");
    //stETH token
    bytes32 public constant ST_ETH_TOKEN = keccak256("ST_ETH_TOKEN");
    //cbETH token
    bytes32 public constant CB_ETH_TOKEN = keccak256("CB_ETH_TOKEN");

    //contracts
    bytes32 public constant LRT_ORACLE = keccak256("LRT_ORACLE");
    bytes32 public constant LRT_DEPOSIT_POOL = keccak256("LRT_DEPOSIT_POOL");
    bytes32 public constant EIGEN_STRATEGY_MANAGER = keccak256("EIGEN_STRATEGY_MANAGER");

    //Roles
20    bytes32 public constant MANAGER = keccak256("MANAGER");

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConstants.sol#L7-L20

## [G-12] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements

Saves 5 gas per iteration

```solidity

file: LRTDepositPool.sol

85     unchecked {
86                ++i;
            }

172    unchecked {
173                ++i;
            }
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L86 
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L173

```solidity

file: NodeDelegator.sol


112  unchecked {
    113                ++i;
            }
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L113

```solidity

file: LRTOracle.sol

73        unchecked {
74                ++asset_idx;
            }

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L73

## [G-13]  Use uint256(1)/uint256(2) instead for true and false boolean states
If you don’t use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

```solidity

file: LRTConfig.sol

85  isSupportedAsset[asset] = true;

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L85

## [G-14] Loop best practice to save gas

```solidity

file: LRTDepositPool.sol

85   unchecked {
                ++i;
            }

172   unchecked {
                ++i;
            }
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L85
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L172

```solidity

file: NodeDelegator.sol

112   unchecked {
                ++i;
            }

73  unchecked {
                ++asset_idx;
            }
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L112
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L73

## [G-15 ] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: LRTDepositPool.sol

83  assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
84   assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L83-L84

```solidity

file: LRTOracle.sol

71  totalETHInPool += totalAssetAmt * assetER;
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L71


## [G‑16] Using calldata instead of memory for read-only arguments in external functions saves gas
When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas

```solidity

file: LRTConfig.sol

135    function getSupportedAssetList() external view override returns (address[] memory) {
        return supportedAssetList;
    }

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L135

```solidity

file: LRTDepositPool.sol

62  function getNodeDelegatorQueue() external view override returns (address[] memory) {
        return nodeDelegatorQueue;
    }
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L62

## [G-17] Use assembly to emit events
We can use assembly to emit events efficiently by utilizing scratch space and the free memory pointer. This will allow us to potentially avoid memory expansion costs. Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

```solidity

file: LRTConfig.sol

88  emit AddedNewSupportedAsset(asset, depositLimit);

103   emit AssetDepositLimitUpdate(asset, depositLimit);

162   emit SetToken(key, val);

178  emit SetContract(key, val);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L88
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L103
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L162
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L178

```solidity

file: LRTDepositPool.sol

37  emit UpdatedLRTConfig(lrtConfigAddr);

143   emit AssetDeposit(asset, depositAmount, rsethAmountMinted);

171  emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]);

204    emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L37
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L143
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L171
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L204

```solidity

file: NodeDelegator.sol

32   emit UpdatedLRTConfig(lrtConfigAddr);

65    emit AssetDepositIntoStrategy(asset, strategy, balance);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L32
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L65

```solidity

file: LRTOracle.sol

34   emit UpdatedLRTConfig(lrtConfigAddr);

98  emit AssetPriceOracleUpdate(asset, priceOracle);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L34
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L98

```solidity

file: RSETH.sol

21  emit UpdatedLRTConfig(lrtConfigAddr);

76   emit UpdatedLRTConfig(_lrtConfig);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L21
https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L76

```solidity

file: oracles/ChainlinkPriceOracle.sol

31      emit UpdatedLRTConfig(lrtConfig_);

48   emit AssetPriceFeedUpdate(asset, priceFeed);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L31
https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L48

```solidity

file: utils/LRTConfigRoleChecker.sol

50  emit UpdatedLRTConfig(lrtConfigAddr);

```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConfigRoleChecker.sol#L50