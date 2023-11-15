## \[L-01\] Stop Using Solidity's transfer() Now

**Proof Of Concept**  
[https://consensys.io/diligence/blog/2019/09/stop-using-soliditys-transfer-now/\[\](https://github.com/Tapioca-DAO/tapioca-bar-audit/blob/master/contracts/markets/singularity/SGLCommon.sol#L215)](https://consensys.io/diligence/blog/2019/09/stop-using-soliditys-transfer-now/%5B%5D%28https://github.com/Tapioca-DAO/tapioca-bar-audit/blob/master/contracts/markets/singularity/SGLCommon.sol#L215%29 "https://consensys.io/diligence/blog/2019/09/stop-using-soliditys-transfer-now/%5B%5D(https://github.com/Tapioca-DAO/tapioca-bar-audit/blob/master/contracts/markets/singularity/SGLCommon.sol#L215)")

```
if (!IERC20(asset).transfer(nodeDelegator, amount)) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L194

```
if (!IERC20(asset).transfer(lrtDepositPool, amount)) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L86

## \[L-02\] Function Calls in Loop Could Lead to Denial of Service

Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to the gas limitations or failed transactions. Here are some of the instances entailed:

```
function getAssetBalances()
        external
        view
        override
        returns (address[] memory assets, uint256[] memory assetBalances)
    {
        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

        (IStrategy[] memory strategies,) =
            IEigenStrategyManager(eigenlayerStrategyManagerAddress).getDeposits(address(this));

        uint256 strategiesLength = strategies.length;
        assets = new address[](strategiesLength);
        assetBalances = new uint256[](strategiesLength);

        for (uint256 i = 0; i < strategiesLength;) {
            assets[i] = address(IStrategy(strategies[i]).underlyingToken());
            assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
            unchecked {
                ++i;
            }
        }
    }
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L94

Consider bounding the loop where possible to avoid unnecessary gas wastage and denial of service.

## \[L-03\] Insufficient coverage

Description  
The test coverage rate of the project is ~98%. Testing all functions is best practice in terms of security criteria.

Due to its capacity, test coverage is expected to be 100%.

## \[L-04\] Keccak Constant values should used to immutable rather than constant

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.

Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

&nbsp;

```
bytes32 public constant R_ETH_TOKEN = keccak256("R_ETH_TOKEN");
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

https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConstants.sol#L7C4-L19C60

# Non- Critical

## \[N-01\] Use underscore for function parameter for better readability

```
function addNewSupportedAsset(address asset, uint256 depositLimit) external onlyRole(LRTConstants.MANAGER) {
        _addNewSupportedAsset(asset, depositLimit);
    }
```

> all contracts have this issue

## \[N-02\] Use a single file for all system-wide constants

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

constants.sol  
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

## \[N-03\] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## \[N-04\] NatSpec comments should be increased in contracts

> all contest

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity Official documentation

in complext project such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.  
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html



## \[S-01\] You can explain the operation of critical functions in NatSpec with an infographic.