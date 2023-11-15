# Gas Report

## Summary

Total **27 instances** over **5 issues**with **27395** gas saved:

|ID|Issue|Instances|Gas|
|:--:|:---|:--:|:--:|
| [[G-01]](#g-01-using-private-for-constants-saves-gas) | Using `private` for constants saves gas | 8 | 27248 |
| [[G-02]](#g-02-do-not-cache-state-variables-that-are-used-only-once) | Do not cache state variables that are used only once | 1 | 3 |
| [[G-03]](#g-03-initializers-can-be-marked-as-payable-to-save-deployment-gas) | Initializers can be marked as payable to save deployment gas | 6 | 126 |
| [[G-04]](#g-04-using-assembly-to-check-for-zero-can-save-gas) | Using assembly to check for zero can save gas | 3 | 18 |
| [[G-05]](#g-05-newer-versions-of-solidity-are-more-gas-efficient) | Newer versions of solidity are more gas efficient | 9 | - |

## Gas Optimizations

### [G-01] Using `private` for constants saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

There are 8 instances:

- *RSETH.sol* ( [21](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/RSETH.sol#L21) ):

```solidity
21:     bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```

- *LRTConstants.sol* ( [7](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/utils/LRTConstants.sol#L7), [9](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/utils/LRTConstants.sol#L9), [11](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/utils/LRTConstants.sol#L11), [14](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/utils/LRTConstants.sol#L14), [15](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/utils/LRTConstants.sol#L15), [16](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/utils/LRTConstants.sol#L16), [19](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/utils/LRTConstants.sol#L19) ):

```solidity
7:     bytes32 public constant R_ETH_TOKEN = keccak256("R_ETH_TOKEN");

9:     bytes32 public constant ST_ETH_TOKEN = keccak256("ST_ETH_TOKEN");

11:     bytes32 public constant CB_ETH_TOKEN = keccak256("CB_ETH_TOKEN");

14:     bytes32 public constant LRT_ORACLE = keccak256("LRT_ORACLE");

15:     bytes32 public constant LRT_DEPOSIT_POOL = keccak256("LRT_DEPOSIT_POOL");

16:     bytes32 public constant EIGEN_STRATEGY_MANAGER = keccak256("EIGEN_STRATEGY_MANAGER");

19:     bytes32 public constant MANAGER = keccak256("MANAGER");
```

### [G-02] Do not cache state variables that are used only once

It's cheaper to access the state variable directly if it is accessed only once. This can save the 3 gas cost of the extra stack allocation.

There is 1 instance:

- *LRTDepositPool.sol* ( [193](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/LRTDepositPool.sol#L193) ):

```solidity
193:         address nodeDelegator = nodeDelegatorQueue[ndcIndex];
```

### [G-03] Initializers can be marked as payable to save deployment gas

Payable functions cost less gas to execute, because the compiler does not have to add extra checks to ensure that no payment is provided. Initializers can be safely marked as payable, because only the deployer or the factory contract would call the function without carrying any funds.

<details>
<summary>There are 6 instances (click to show):</summary>

- *LRTConfig.sol* ( [41-50](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/LRTConfig.sol#L41-L50) ):

```solidity
41:     function initialize(
42:         address admin,
43:         address stETH,
44:         address rETH,
45:         address cbETH,
46:         address rsETH_
47:     )
48:         external
49:         initializer
50:     {
```

- *LRTDepositPool.sol* ( [31](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/LRTDepositPool.sol#L31) ):

```solidity
31:     function initialize(address lrtConfigAddr) external initializer {
```

- *LRTOracle.sol* ( [29](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/LRTOracle.sol#L29) ):

```solidity
29:     function initialize(address lrtConfigAddr) external initializer {
```

- *NodeDelegator.sol* ( [26](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/NodeDelegator.sol#L26) ):

```solidity
26:     function initialize(address lrtConfigAddr) external initializer {
```

- *RSETH.sol* ( [32](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/RSETH.sol#L32) ):

```solidity
32:     function initialize(address admin, address lrtConfigAddr) external initializer {
```

- *ChainlinkPriceOracle.sol* ( [27](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/oracles/ChainlinkPriceOracle.sol#L27) ):

```solidity
27:     function initialize(address lrtConfig_) external initializer {
```

</details>

### [G-04] Using assembly to check for zero can save gas

Using assembly to check for zero can save gas by allowing more direct access to the evm and reducing some of the overhead associated with high-level operations in solidity.

There are 3 instances:

- *LRTDepositPool.sol* ( [129](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/LRTDepositPool.sol#L129) ):

```solidity
129:         if (depositAmount == 0) {
```

- *LRTOracle.sol* ( [56](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/LRTOracle.sol#L56) ):

```solidity
56:         if (rsEthSupply == 0) {
```

- *UtilLib.sol* ( [12](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/utils/UtilLib.sol#L12) ):

```solidity
12:         if (address_ == address(0)) revert ZeroAddressNotAllowed();
```

### [G-05] Newer versions of solidity are more gas efficient

The solidity language continues to pursue more efficient gas optimization schemes. Adopting a [newer version of solidity](https://github.com/ethereum/solc-js/tags) can be more gas efficient.

<details>
<summary>There are 9 instances (click to show):</summary>

- *LRTConfig.sol* ( [2](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/LRTConfig.sol#L2) ):

```solidity
2: pragma solidity 0.8.21;
```

- *LRTDepositPool.sol* ( [2](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/LRTDepositPool.sol#L2) ):

```solidity
2: pragma solidity 0.8.21;
```

- *LRTOracle.sol* ( [2](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/LRTOracle.sol#L2) ):

```solidity
2: pragma solidity 0.8.21;
```

- *NodeDelegator.sol* ( [2](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/NodeDelegator.sol#L2) ):

```solidity
2: pragma solidity 0.8.21;
```

- *RSETH.sol* ( [2](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/RSETH.sol#L2) ):

```solidity
2: pragma solidity 0.8.21;
```

- *ChainlinkPriceOracle.sol* ( [2](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/oracles/ChainlinkPriceOracle.sol#L2) ):

```solidity
2: pragma solidity 0.8.21;
```

- *LRTConfigRoleChecker.sol* ( [2](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/utils/LRTConfigRoleChecker.sol#L2) ):

```solidity
2: pragma solidity 0.8.21;
```

- *LRTConstants.sol* ( [2](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/utils/LRTConstants.sol#L2) ):

```solidity
2: pragma solidity 0.8.21;
```

- *UtilLib.sol* ( [2](https://github.com/code-423n4/2023-11-kelp/blob/0f4f79fe8a58222320a2b7b4d0fdc5663a333b0e/src/utils/UtilLib.sol#L2) ):

```solidity
2: pragma solidity 0.8.21;
```

</details>
