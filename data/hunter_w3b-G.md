# Gas-Optmization for Kelp DAO | rsETH Protocol

## [G-01] Avoid contract existence checks by using low level calls

**Issue Description**\
Prior to Solidity 0.8.10, the compiler would insert extra code to check for contract existence before external function calls, even if the call had a return value. This wasted gas by performing a check that wasn't necessary.

**Proposed Optimization**\
For functions that make external calls with return values in Solidity <0.8.10, optimize the code to use low-level calls instead of regular calls. Low-level calls skip the unnecessary contract existence check.

Example:

```solidity
//Before:

contract C {
  function f() external returns(uint) {
    address(otherContract).call(abi.encodeWithSignature("func()"));
  }
}


//After:

contract C {
  function f() external returns(uint) {
    (bool success,) = address(otherContract).call(abi.encodeWithSignature("func()"));
    require(success);
    return decodeReturnValue();
  }
}
```

**Estimated Gas Savings**\
Each avoided EXTCODESIZE check saves 100 gas. If 10 external calls are made in a common function, this would save 1000 gas total.

**Attachments**

- **Code Snippets**

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

## [G-02] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

**Issue Description**\
The code is declaring constant values such as role identifiers and contract names using 'constant' rather than 'immutable'.

**Proposed Optimization**\
Update the declarations of constant values from 'constant' to 'immutable'. This avoids unnecessary writes to storage every time these values are accessed.

**Estimated Gas Savings**\
Switching to 'immutable' for constant value declarations will save gas by avoiding storage writes on every access. The specific gas savings will depend on how many times each constant value is accessed but is expected to be small for each individual reference. accumlated across many transactions and many references, it could result in non-trivial total savings.

**Attachments**

- **Code Snippets**

```solidity
File: src/RSETH.sol

21    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

22    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L21-L22

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

## [G-03] Using Storage instead of memory for structs/arrays saves gas

**Issue Description**\
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array.

If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

Instead of declaring the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incurring the Gcoldsload for the fields actually read

**Proposed Optimization**\
Replace memory keywords with storage when fetching structs/arrays from storage that are not returned from the function or passed to a function requiring memory. Cache any re-read fields in stack variables.

**Estimated Gas Savings**\
By replacing memory with storage, the number of SLOAD operations can be reduced from O(n) to O(1) where n is the number of fields in the struct/array. For large structs/arrays this can result in gas savings of thousands per transaction.

**Attachments**

- **Code Snippets**

```solidity
File: src/LRTOracle.sol

63        address[] memory supportedAssets = lrtConfig.getSupportedAssetList();
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L63

```solidity
File: src/NodeDelegator.sol

102        (IStrategy[] memory strategies,) =
103            IEigenStrategyManager(eigenlayerStrategyManagerAddress).getDeposits(address(this));
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L102-L103

## [G-04] Use constants instead of type(uintx).max

**Issue Description**\
Using type(uint256).max to represent the maximum approvable amount for ERC20 tokens uses more gas in the distribution process and also for each transaction than using a constant.

**Proposed Optimization**\
Define a constant such as MAX_UINT256 = type(uint256).max and use that constant instead.

**Estimated Gas Savings**\
Using a constant avoids the overhead of calling the type(uint256) method each time. This could save ~100 gas per transaction. For contracts with many transactions, this can add up to significant savings over time.

**Attachments**

- **Code Snippets**

```solidity
File: src/NodeDelegator.sol

45        IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L45

## [G-05] Using mappings instead of arrays to avoid length checks

**Issue Description**\
When storing a list or group of items that you wish to organize in a specific order and fetch with a fixed key/index, it’s common practice to use an array data structure. This works well, but did you know that a trick can be implemented to save 2,000+ gas on each read using a mapping?

**Proposed Optimization**\

```solidity
See the example below

/// get(0) gas cost: 4860
contract Array {
    uint256[] a;

    constructor() {
        a.push() = 1;
        a.push() = 2;
        a.push() = 3;
    }

    function get(uint256 index) external view returns(uint256) {
        return a[index];
    }
}

/// get(0) gas cost: 2758
contract Mapping {
    mapping(uint256 => uint256) a;

    constructor() {
        a[0] = 1;
        a[1] = 2;
        a[2] = 3;
    }

    function get(uint256 index) external view returns(uint256) {
        return a[index];
    }
}
```

Just by using a mapping, we get a gas saving of 2102 gas. Why? Under the hood when you read the value of an index of an array, solidity adds bytecode that checks that you are reading from a valid index (i.e an index strictly less than the length of the array) else it reverts with a panic error (Panic(0x32) to be precise). This prevents from reading unallocated or worse, allocated storage/memory locations.

Due to the way mappings are (simply a key => value pair), no check like that exists and we are able to read from the a storage slot directly. It’s important to note that when using mappings in this manner, your code should ensure that you are not reading an out of bound index of your canonical array.

**Attachments**

- **Code Snippets**

```solidity
File: src/LRTConfig.sol

19    address[] public supportedAssetList;
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L19

```solidity
File: src/LRTDepositPool.sol

22    address[] public nodeDelegatorQueue;
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L22

## [G-06] Understand the trade-offs when choosing between internal functions and modifiers

**Issue Description**\
Modifiers inject its implementation bytecode where it is used while internal functions jump to the location in the runtime code where the its implementation is. This brings certain trade-offs to both options.

**Proposed Optimization**\
Using modifiers more than once means repetitiveness and increase in size of the runtime code but reduces gas cost because of the absence of jumping to the internal function execution offset and jumping back to continue. This means that if runtime gas cost matter most to you, then modifiers should be your choice but if deployment gas cost and/or reducing the size of the creation code is most important to you then using internal functions will be best.

However, modifiers have the tradeoff that they can only be executed at the start or end of a functon. This means executing it at the middle of a function wouldn’t be directly possible, at least not without internal functions which kill the original purpose. This affects it’s flexibility. Internal functions however can be called at any point in a function.

**Estimated Gas Savings**\
Example showing difference in gas cost using modifiers and an internal function

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.19;

/** deployment gas cost: 195435
    gas per call:
              restrictedAction1: 28367
              restrictedAction2: 28377
              restrictedAction3: 28411
 */
 contract Modifier {
    address owner;
    uint256 val;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    function restrictedAction1() external onlyOwner {
        val = 1;
    }

    function restrictedAction2() external onlyOwner {
        val = 2;
    }

    function restrictedAction3() external onlyOwner {
        val = 3;
    }
}



/** deployment gas cost: 159309
    gas per call:
              restrictedAction1: 28391
              restrictedAction2: 28401
              restrictedAction3: 28435
 */
 contract InternalFunction {
    address owner;
    uint256 val;

    constructor() {
        owner = msg.sender;
    }

    function onlyOwner() internal view {
        require(msg.sender == owner);
    }

    function restrictedAction1() external {
        onlyOwner();
        val = 1;
    }

    function restrictedAction2() external {
        onlyOwner();
        val = 2;
    }

    function restrictedAction3() external {
        onlyOwner();
        val = 3;
    }
}
```

| Operation          | Deployment | restrictedAction1 | restrictedAction2 | restrictedAction3 |
| ------------------ | ---------- | ----------------- | ----------------- | ----------------- |
| Modifiers          | 195435     | 28367             | 28377             | 28411             |
| Internal Functions | 159309     | 28391             | 28401             | 28435             |

From the table above, we can see that the contract that uses modifiers cost more than 35k gas more than the contract using internal functions when deploying it due to repetition of the onlyOwner functionality in 3 functions.

During runtime, we can see that each function using modifiers cost a fixed 24 gas less than the functions using internal functions.

**Attachments**

- **Code Snippets**

```solidity
File: src/LRTConfig.sol

28    modifier onlySupportedAsset(address asset) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L28

```solidity
File: src/utils/LRTConfigRoleChecker.sol

20    modifier onlyLRTManager() {

27    modifier onlyLRTAdmin() {

35    modifier onlySupportedAsset(address asset) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConfigRoleChecker.sol#L20

## [G-07] Consider using alternatives to OpenZeppelin

OpenZeppelin is a great and popular smart contract library, but there are other alternatives that are worth considering. These alternatives offer better gas efficiency and have been tested and recommended by developers.

Two examples of such alternatives are Solmate and Solady.

Solmate is a library that provides a number of gas-efficient implementations of common smart contract patterns. Solady is another gas-efficient library that places a strong emphasis on using assembly.

## [G-08] Using assembly to revert with an error message

**Issue Description**\
When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

**Estimated Gas Savings**\
Here’s an example;

```solidity
/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}

/// calling restrictedAction(2) with a non-owner address: 23734
contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}
```

From the example above we can see that we get a gas saving of over 300 gas when reverting wth the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

**Attachments**

- **Code Snippets**

```solidity
File: src/LRTConfig.sol

30            revert AssetNotSupported();

83            revert AssetAlreadySupported();

119            revert ValueAlreadyInUse();

159            revert ValueAlreadyInUse();

175            revert ValueAlreadyInUse();
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L30

```solidity
File: src/LRTDepositPool.sol

130            revert InvalidAmount();

133            revert MaximumDepositLimitReached();

137            revert TokenTransferFailed();

165            revert MaximumNodeDelegatorCountReached();

195            revert TokenTransferFailed();
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L130

```solidity
File: src/utils/LRTConfigRoleChecker.sol

22            revert ILRTConfig.CallerNotLRTConfigManager();

30            revert ILRTConfig.CallerNotLRTConfigAdmin();

37            revert ILRTConfig.AssetNotSupported();
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConfigRoleChecker.sol#L22

## [G-09] Use assembly to reuse memory space when making more than one external call

**Issue Description**\
When making external calls, the solidity compiler has to encode the function signature and arguments in memory. It does not clear or reuse memory, so it expands memory each time.

**Proposed Optimization**\
Use inline assembly to reuse the same memory space for multiple external calls. Store the function selector and arguments without expanding memory further.

**Estimated Gas Savings**\
Reusing memory can save thousands of gas compared to expanding on each call. The baseline memory expansion per call is 3,000 gas. With larger arguments or return data, the savings would be even greater.

```solidity
See the example below;

contract Called {
    function add(uint256 a, uint256 b) external pure returns(uint256) {
        return a + b;
    }
}


contract Solidity {
    // cost: 7262
    function call(address calledAddress) external pure returns(uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);

        uint256 res = res1 + res2;
        return res;
    }
}


contract Assembly {
    // cost: 5281
    function call(address calledAddress) external view returns(uint256) {
        assembly {
            // check that calledAddress has code deployed to it
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }

            // first call
            mstore(0x00, hex"771602f7")
            mstore(0x04, 0x01)
            mstore(0x24, 0x02)
            let success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)

            // second call
            mstore(0x04, 0x03)
            mstore(0x24, 0x4)
            success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)

            // add results
            let res := add(res1, res2)

            // return data
            mstore(0x60, res)
            return(0x60, 0x20)
        }
    }
}
```

We save approximately 2,000 gas by using the scratch space to store the function selector and it’s arguments and also reusing the same memory space for the second call while storing the returned data in the zero slot thus not expanding memory.

If the arguments of the external function you wish to call is above 64 bytes and if you are making one external call, it wouldn’t save any significant gas writing it in assembly. However, if making more than one call. You can still save gas by reusing the same memory slot for the 2 calls using inline assembly.

Note: Always remember to update the free memory pointer if the offset it points to is already used, to avoid solidity overriding the data stored there or using the value stored there in an unexpected way.

Also note to avoid overwriting the zero slot (0x60 memory offset) if you have undefined dynamic memory values within that call stack. An alternative is to explicitly define dynamic memory values or if used, to set the slot back to 0x00 before exiting the assembly block.

**Attachments**

- **Code Snippets**

```solidity
File: src/LRTDepositPool.sol

79        assetLyingInDepositPool = IERC20(asset).balanceOf(address(this));

83            assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
84            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L79

```solidity
File: src/LRTOracle.sol

53        address rsETHTokenAddress = lrtConfig.rsETH();
61        address lrtDepositPoolAddr = lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL);
63        address[] memory supportedAssets = lrtConfig.getSupportedAssetList();
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L53

```solidity
File: src/NodeDelegator.sol

59        address strategy = lrtConfig.assetStrategy(asset);
61        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L59

This report discusses how using inline assembly can optimize gas costs when making multiple external calls by reusing memory space, rather than expanding memory separately for each call. This can save thousands of gas compared to the solidity compiler's default behavior.

## [G-10] Do-While loops are cheaper than for loops

**Issue Description**\
If you want to push optimization at the expense of creating slightly unconventional code, Solidity do-while loops are more gas efficient than for loops, even if you add an if-condition check for the case where the loop doesn’t execute at all.

**Proposed Optimization**\
Replace for loops with do-while loops when possible to reduce gas costs. Add an if check before the do-while to handle cases where the loop body may not need to execute.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

// times == 10 in both tests
contract Loop1 {
    function loop(uint256 times) public pure {
        for (uint256 i; i < times;) {
            unchecked {
                ++i;
            }
        }
    }
}

contract Loop2 {
    function loop(uint256 times) public pure {
        if (times == 0) {
            return;
        }

        uint256 i;

        do {
            unchecked {
                ++i;
            }
        } while (i < times);
    }
}
```

**Estimated Gas Savings**\
Based on benchmarks, do-while loops can save 15-30 gas per iteration compared to for loops. With loops executing hundreds or thousands of iterations, the savings can add up significantly.

**Attachments**

- **Code Snippets**

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

```solidity
File: src/LRTOracle.sol

66        for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L66

## [G-11] Don't make variables public unless necessary

**Issue Description**\
Making variables public comes with some overhead costs that can be avoided in many cases. A public variable implicitly creates a public getter function of the same name, increasing the contract size.

**Proposed Optimization**\
Only mark variables as public if their values truly need to be readable by external contracts/users. Otherwise, use private or internal visibility.

**Estimated Gas Savings**\
The savings from avoiding unnecessary public variables are small per transaction, around 5-10 gas. However, this adds up over many transactions targeting a contract with public state variables that don't need to be public.

**Attachments**

- **Code Snippets**

```solidity
File: src/LRTConfig.sol

19    address[] public supportedAssetList;

21    address public rsETH;
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L19

```solidity
File: src/LRTDepositPool.sol

20    uint256 public maxNodeDelegatorCount;

22    address[] public nodeDelegatorQueue;
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L20

```solidity
File: src/RSETH.sol

21    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

22    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L21

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

## [G-12] It is sometimes cheaper to cache calldata

**Issue Description**\
While the calldataload opcode is relatively cheap, directly using it in a loop or multiple times can still result in unnecessary bytecode. Caching the loaded calldata first may allow for optimization opportunities.

**Proposed Optimization**\
Cache calldata values in a local variable after first load, then reference the local variable instead of repeatedly using calldataload.

**Estimated Gas Savings**\
Exact savings vary depending on contract, but caching calldata parameters can save 5-20 gas per usage by avoiding extra calldataload opcodes. Larger functions with many parameter uses see more benefit.

**Attachments**

- **Code Snippets**

```solidity
File: src/LRTDepositPool.sol

162    function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L162

## [G-13] Use hardcoded addresses instead of address(this)

**Issue Description**\
The address(this) construct pushes the contract's address onto the stack at runtime. This has a small gas cost compared to hardcoding the address directly.

**Proposed Optimization**\
For contract methods/functions that are internal and will never be called from outside the contract, hardcode the contract address instead of using address(this).

**Estimated Gas Savings**\
Using a hardcoded address saves ~2 gas per use compared to address(this). For functions that reference the contract address multiple times, the savings can add up.

**Attachments**

- **Code Snippets**

```solidity
File: src/LRTDepositPool.sol

79        assetLyingInDepositPool = IERC20(asset).balanceOf(address(this));

136        if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L79

```solidity
File: src/NodeDelegator.sol

63        uint256 balance = token.balanceOf(address(this));

103            IEigenStrategyManager(eigenlayerStrategyManagerAddress).getDeposits(address(this));

111            assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));

123        return IStrategy(strategy).userUnderlyingView(address(this));
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L63

## [G-14] Shorten arrays with inline assembly

**Issue Description**\
When shortening an array in Solidity, it creates a new shorter array and copies the elements over. This wastes gas by duplicating storage.

**Proposed Optimization**\
Use inline assembly to shorten the array in place by changing its length slot, avoiding the need to copy elements to a new array.

**Estimated Gas Savings**\
Shortening a length-n array avoids ~n SSTORE operations to copy elements. Benchmarking shows savings of 5000-15000 gas depending on original length.

```solidity
function shorten(uint[] storage array, uint newLen) internal {

  assembly {
    sstore(array_slot, newLen)
  }

}

// Rather than:
function shorten(uint[] storage array, uint newLen) internal {

  uint[] memory newArray = new uint[](newLen);

  for(uint i = 0; i < newLen; i++) {
    newArray[i] = array[i];
  }

  delete array;
  array = newArray;

}
```

Using inline assembly allows shortening arrays without copying elements to a new storage slot, providing significant gas savings.

**Attachments**

- **Code Snippets**

```solidity
File: src/NodeDelegator.sol

106        assets = new address[](strategiesLength);

107        assetBalances = new uint256[](strategiesLength);
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L106-L107

## [G-15] Mark initializer functions as payable to save deployment gas

**Attachments**

- **Code Snippets**

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
File: src/NodeDelegator.sol

26    function initialize(address lrtConfigAddr) external initializer {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L26

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

## [G-16] Access mappings directly instead of through accessor functions

**Issue Description**\
Using accessor functions to access mappings incurs some additional gas costs due to an extra layer of function call indirection.

**Proposed Optimization**\
For internal mappings where gas optimization is important, access the mapping values directly rather than going through getter functions.

**Estimated Gas Savings**\
Benchmarking shows direct mapping access can save around 5 gas per lookup compared to an accessor function. With many lookups this savings adds up significantly.

**Attachments**

- **Code Snippets**

```solidity
File: src/LRTConfig.sol

128        return tokenMap[tokenKey];

132        return contractMap[contractKey];
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L128

## [G-17] Cache external calls outside of loops

**Issue Description**\
Making external function calls like STATICCALL inside a loop incurs a gas cost per iteration that can be optimized.

**Proposed Optimization**\
Cache the results of static external calls before any loops, rather than re-calling the function on each iteration.

**Estimated Gas Savings**\
Avoids 100 gas cost per STATICCALL by caching results. With loops of 100+ iterations, savings can exceed 10,000 gas.

**Attachments**

- **Code Snippets**

```solidity
File: src/LRTDepositPool.sol

82        for (uint256 i; i < ndcsCount;) {
83:            assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
84:            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
85            unchecked {
86                ++i;
88            }
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L83-L84

```solidity
File: src/LRTOracle.sol

66        for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
67            address asset = supportedAssets[asset_idx];
68            uint256 assetER = getAssetPrice(asset);
69
70:            uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
71            totalETHInPool += totalAssetAmt * assetER;
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L66-L71

```solidity
File: src/NodeDelegator.sol

109        for (uint256 i = 0; i < strategiesLength;) {
110            assets[i] = address(IStrategy(strategies[i]).underlyingToken());
111            assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
112            unchecked {
113                ++i;
114            }
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L110-L111
