# Gas-Optimization

## [G-01] Use do while  loops instead of for loops

```solidity
File: src/LRTOracle.sol

66        for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L66

```solidity
File: src/LRTDepositPool.sol

82        for (uint256 i; i < ndcsCount;) {

168        for (uint256 i; i < length;) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L82

```solidity
File: src/NodeDelegator.sol

109        for (uint256 i = 0; i < strategiesLength;) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L109

## [G-02] Using mappings instead of arrays to avoid length checks

```solidity
File: src/LRTDepositPool.sol

22    address[] public nodeDelegatorQueue;
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L22

```solidity
File: src/LRTConfig.sol

19    address[] public supportedAssetList;
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L19

## [G-03] Don't make variables public unless necessary

```solidity
File: src/RSETH.sol

21    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

22    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L21

```solidity
File: src/LRTDepositPool.sol

20    uint256 public maxNodeDelegatorCount;

22    address[] public nodeDelegatorQueue;
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L20

```solidity
File: src/LRTConfig.sol

19    address[] public supportedAssetList;

21    address public rsETH;
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L19

```solidity
File: src/utils/LRTConfigRoleChecker.sol

14    ILRTConfig public lrtConfig;
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConfigRoleChecker.sol#L14

```solidity
File: src/utils/LRTConstants.sol

7    bytes32 public constant R_ETH_TOKEN = keccak256("R_ETH_TOKEN");

9    bytes32 public constant ST_ETH_TOKEN = keccak256("ST_ETH_TOKEN");

11    bytes32 public constant CB_ETH_TOKEN = keccak256("CB_ETH_TOKEN");

14    bytes32 public constant LRT_ORACLE = keccak256("LRT_ORACLE");

15    bytes32 public constant LRT_DEPOSIT_POOL = keccak256("LRT_DEPOSIT_POOL");

16    bytes32 public constant EIGEN_STRATEGY_MANAGER = keccak256("EIGEN_STRATEGY_MANAGER");

19    bytes32 public constant MANAGER = keccak256("MANAGER");
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConstants.sol#L7

## [G-04] Using Storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from
storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array.

If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to
be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
File: src/NodeDelegator.sol

102        (IStrategy[] memory strategies,) =
103            IEigenStrategyManager(eigenlayerStrategyManagerAddress).getDeposits(address(this));
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L102-L103

```solidity
File: src/LRTOracle.sol

63        address[] memory supportedAssets = lrtConfig.getSupportedAssetList();
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L63

## [G-05] Use hardcoded addresses instead of address(this)

```solidity
File: src/NodeDelegator.sol

63        uint256 balance = token.balanceOf(address(this));

103            IEigenStrategyManager(eigenlayerStrategyManagerAddress).getDeposits(address(this));

111            assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));

123        return IStrategy(strategy).userUnderlyingView(address(this));
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L63

```solidity
File: src/LRTDepositPool.sol

79        assetLyingInDepositPool = IERC20(asset).balanceOf(address(this));

136        if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L79

## [G-06] Mark initializer functions as payable to save deployment gas

```solidity
File: src/NodeDelegator.sol

26    function initialize(address lrtConfigAddr) external initializer {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L26

```solidity
File: src/LRTConfig.sol

41    function initialize(
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L41-L68

```solidity
File: src/LRTDepositPool.sol

31    function initialize(address lrtConfigAddr) external initializer {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L31

```solidity
File: src/LRTOracle.sol

29    function initialize(address lrtConfigAddr) external initializer {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L29

```solidity
File: src/RSETH.sol

32    function initialize(address admin, address lrtConfigAddr) external initializer {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L32

```solidity
File: src/oracles/ChainlinkPriceOracle.sol

27    function initialize(address lrtConfig_) external initializer {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L27

## [G-07] Avoid contract existence checks by using low level calls

```solidity
File: src/LRTDepositPool.sol

79        assetLyingInDepositPool = IERC20(asset).balanceOf(address(this));

83            assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);

84            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);

105        address lrtOracleAddress = lrtConfig.getContract(LRTConstants.LRT_ORACLE);

109        rsethAmountToMint = (amount * lrtOracle.getAssetPrice(asset)) / lrtOracle.getRSETHPrice();

136        if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) {

156        IRSETH(rsethToken).mint(msg.sender, rsethAmountToMint);

194        if (!IERC20(asset).transfer(nodeDelegator, amount)) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L79

```solidity
File: src/oracles/ChainlinkPriceOracle.sol

38        return AggregatorInterface(assetPriceFeed[asset]).latestAnswer();
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L38

```solidity
File:  src/utils/LRTConfigRoleChecker.sol

21        if (!IAccessControl(address(lrtConfig)).hasRole(LRTConstants.MANAGER, msg.sender)) {

29        if (!IAccessControl(address(lrtConfig)).hasRole(DEFAULT_ADMIN_ROLE, msg.sender)) {

36        if (!IAccessControl(address(lrtConfig)).hasRole(DEFAULT_ADMIN_ROLE, msg.sender)) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConfigRoleChecker.sol#L21

```solidity
File: src/LRTOracle.sol

46        return IPriceFetcher(assetPriceOracle[asset]).getAssetPrice(asset);

54        uint256 rsEthSupply = IRSETH(rsETHTokenAddress).totalSupply();

70            uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L46

```solidity
File: src/NodeDelegator.sol

44        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

45        IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);

59        address strategy = lrtConfig.assetStrategy(asset);

61        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

67        IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);

84        address lrtDepositPool = lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL);

103            IEigenStrategyManager(eigenlayerStrategyManagerAddress).getDeposits(address(this));
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L44

## [G-08] Shorten arrays with inline assembly

```solidity
File: src/NodeDelegator.sol

106        assets = new address[](strategiesLength);

107        assetBalances = new uint256[](strategiesLength);
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L106-L107

## [G-09] Use constants instead of type(uintx).max

```solidity
File: src/NodeDelegator.sol

45        IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L45

## [G-10] It is sometimes cheaper to cache calldata

```solidity
File: src/LRTDepositPool.sol

162    function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L162

## [G-11] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

```solidity
File: src/utils/LRTConstants.sol

7    bytes32 public constant R_ETH_TOKEN = keccak256("R_ETH_TOKEN");

9    bytes32 public constant ST_ETH_TOKEN = keccak256("ST_ETH_TOKEN");

11    bytes32 public constant CB_ETH_TOKEN = keccak256("CB_ETH_TOKEN");

14    bytes32 public constant LRT_ORACLE = keccak256("LRT_ORACLE");

15    bytes32 public constant LRT_DEPOSIT_POOL = keccak256("LRT_DEPOSIT_POOL");

16    bytes32 public constant EIGEN_STRATEGY_MANAGER = keccak256("EIGEN_STRATEGY_MANAGER");

19    bytes32 public constant MANAGER = keccak256("MANAGER");
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConstants.sol#L7

```solidity
File: src/RSETH.sol

21    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

22    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L21-L22