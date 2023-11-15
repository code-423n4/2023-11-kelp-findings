## \[G‑1\] Newer versions of solidity are more gas efficient

The solidity language continues to pursue more efficient gas optimization schemes. Adopting a <ins>newer version of solidity</ins> can be more gas efficient.

```
pragma solidity 0.8.21;
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L2

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L2

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L2

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L2

https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L2

https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L2

https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L2

https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConfigRoleChecker.sol#L2

https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/UtilLib.sol#L2

## \[G‑2\] Initializers can be marked as payable to save deployment gas

Payable functions cost less gas to execute, because the compiler does not have to add extra checks to ensure that no payment is provided. Initializers can be safely marked as payable, because only the deployer or the factory contract would call the function without carrying any funds.

```
function initialize(
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L41

```
function initialize(address lrtConfigAddr) external initializer {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L31

```
function initialize(address lrtConfigAddr) external initializer {
```

&nbsp;https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L29

```
function initialize(address lrtConfigAddr) external initializer {
```

&nbsp;https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L29

```
function initialize(address admin, address lrtConfigAddr) external initializer {
```

&nbsp;https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L32

```
function initialize(address lrtConfig_) external initializer {
```

&nbsp;https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L27

```
function initialize(address admin, address lrtConfigAddr) external initializer {
```

&nbsp;https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L32

## \[G-3\] Extra declaration in named returns that don't use them

Using both named returns and a return statement isn’t necessary, this consumes extra gas as one more variable that is not used is declared .

```
function getTotalAssetDeposits(address asset) public view override returns (uint256 totalAssetDeposit) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L47

```
function getRSETHPrice() external view returns (uint256 rsETHPrice) {
```

&nbsp;https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L52

## \[G-4\] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

```
assetLyingInDepositPool = IERC20(asset).balanceOf(address(this));
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L79

```
if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L136

```
uint256 balance = token.balanceOf(address(this));
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L63

```
IEigenStrategyManager(eigenlayerStrategyManagerAddress).getDeposits(address(this));
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L103

```
assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
```

&nbsp;https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L111

```
return IStrategy(strategy).userUnderlyingView(address(this));
```

&nbsp;https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L123

## \[G‑5\] Using assembly to check for zero can save gas

Using assembly to check for zero can save gas by allowing more direct access to the evm and reducing some of the overhead associated with high-level operations in solidity.

```
if (depositAmount == 0) {
```

```
if (rsEthSupply == 0) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L56

```
if (address_ == address(0)) revert ZeroAddressNotAllowed();
```

&nbsp;https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/UtilLib.sol#L12

## \[G‑6\] Using private rather than public for constants, saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table.

```
bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L21C2-L22C68

```
bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

&nbsp;https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L21C4-L22C68

&nbsp;

## \[G-7\] Reduce the Use of keccak256:

Instead of using `keccak256` for every constant, use `bytes32` directly. This can save gas because you're not hashing a string each time.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConstants.sol

```diff
// SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity 0.8.21;

library LRTConstants {
    // Tokens
-    bytes32 public constant R_ETH_TOKEN = keccak256("R_ETH_TOKEN");
+    bytes32 public constant R_ETH_TOKEN = 0x52920a1e6aa9d41e72d9871166f037388be81880;
-    bytes32 public constant ST_ETH_TOKEN = keccak256("ST_ETH_TOKEN");
+    bytes32 public constant ST_ETH_TOKEN = 0x9c3dd2f5375d0b1f611226b7b8d1dd62a52a86c1;
-    bytes32 public constant CB_ETH_TOKEN = keccak256("CB_ETH_TOKEN");
+    bytes32 public constant CB_ETH_TOKEN = 0x0e39c29064e8de783098c6970700643a76037b7f;

    // Contracts
-    bytes32 public constant LRT_ORACLE = keccak256("LRT_ORACLE");
+    bytes32 public constant LRT_ORACLE = 0x8a0a5319ab3cf98868bd75402e3e71a2be5c11d3;
-    bytes32 public constant LRT_DEPOSIT_POOL = keccak256("LRT_DEPOSIT_POOL");
+    bytes32 public constant LRT_DEPOSIT_POOL = 0xd79e2052ec7f25c0fb9e1c372b34ce1f42f11f2b;
-    bytes32 public constant EIGEN_STRATEGY_MANAGER = keccak256("EIGEN_STRATEGY_MANAGER");
+    bytes32 public constant EIGEN_STRATEGY_MANAGER = 0xf4f73344e17f0751f366275c7ed832808c632f7d;

    // Roles
-    bytes32 public constant MANAGER = keccak256("MANAGER");
+    bytes32 public constant MANAGER = 0xd8e33e8ed9f02ab51f3dd82ff63a2b4bf7fd3d36;
    
}
```

Note: The hexadecimal values are placeholders and should be replaced with the actual values you intend to use. Always ensure that the constants' values are unique and secure.

```
bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L21C2-L22C68

```
bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L21C1-L22C68

## \[G-8\] Use Custom Errors

Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

```diff
function checkNonZeroAddress(address address_) internal pure {
-        if (address_ == address(0)) revert ZeroAddressNotAllowed();

    }
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/UtilLib.sol#L12

```
if (!IAccessControl(address(lrtConfig)).hasRole(LRTConstants.MANAGER, msg.sender)) {
            revert ILRTConfig.CallerNotLRTConfigManager();
        }
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConfigRoleChecker.sol#L21C8-L23C10

```
if (!IAccessControl(address(lrtConfig)).hasRole(DEFAULT_ADMIN_ROLE, msg.sender)) {
            revert ILRTConfig.CallerNotLRTConfigAdmin();
        }
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConfigRoleChecker.sol#L29C8-L31C10

```
if (!lrtConfig.isSupportedAsset(asset)) {
            revert ILRTConfig.AssetNotSupported();
        }
```

## \[G-9\] Use `require` for Preconditions:

Replace custom `UtilLib` checks with `require` statements:

```diff
function initialize(address admin, address lrtConfigAddr) external initializer {
-        UtilLib.checkNonZeroAddress(admin);
-        UtilLib.checkNonZeroAddress(lrtConfigAddr);
+        require(admin != address(0), "Invalid admin address");
+        require(lrtConfigAddr != address(0), "Invalid LRT config address");

    // ... rest of the initialization
}
```

By using `require`, you improve code readability and make it clear that certain conditions must be met for the function to proceed. This is a standard practice in Solidity.

```
UtilLib.checkNonZeroAddress(lrtConfig_);
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L28

```
UtilLib.checkNonZeroAddress(priceFeed);
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L46

## \[G-10\]  Combine Mint and Burn Functions:

If the mint and burn functions have similar logic, consider combining them into a single function:

https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L47C4-L56C6

```
-   function mint(address to, uint256 amount) external onlyRole(MINTER_ROLE) whenNotPaused {
        _mint(to, amount);
    }


    /// @notice Burns rsETH when called by an authorized caller
    /// @param account the account to burn from
    /// @param amount the amount of rsETH to burn
-   function burnFrom(address account, uint256 amount) external onlyRole(BURNER_ROLE) whenNotPaused {
        _burn(account, amount);
    }


+ function manageTokens(address account, uint256 amount, bool isMint) external whenNotPaused {
    require(hasRole(isMint ? MINTER_ROLE : BURNER_ROLE, msg.sender), "Not authorized");
    
    if (isMint) {
        _mint(account, amount);
    } else {
        _burn(account, amount);
    }
}
```

This consolidation can reduce duplicated code and simplify the contract structure. It also makes it easier to maintain and understand.

## \[G‑11\] Save gas by preventing zero amount in mint() and burn()

```
function mint(address to, uint256 amount) external onlyRole(MINTER_ROLE) whenNotPaused {
        _mint(to, amount);
    }


    /// @notice Burns rsETH when called by an authorized caller
    /// @param account the account to burn from
    /// @param amount the amount of rsETH to burn
    function burnFrom(address account, uint256 amount) external onlyRole(BURNER_ROLE) whenNotPaused {
        _burn(account, amount);
    }
```

&nbsp;https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L47C3-L56C6

```
function _mintRsETH(address _asset, uint256 _amount) private returns (uint256 rsethAmountToMint) {
        (rsethAmountToMint) = getRsETHAmountToMint(_asset, _amount);


        address rsethToken = lrtConfig.rsETH();
        // mint rseth for user
        IRSETH(rsethToken).mint(msg.sender, rsethAmountToMint);
    }
```

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L151C4-L157C6

## \[G-12\] Amounts should be checked for 0 before calling a transfer

It is considered a best practice to verify for zero values before executing any transfers within smart contract functions. This approach helps prevent unnecessary external calls and can effectively reduce gas costs.

This practice is particularly crucial when dealing with the transfer of tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can determine whether a value is zero by utilizing the == operator. Here's an example demonstrating how to check for a zero value before performing a transfer:

```
//example 
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```

```
if (!IERC20(asset).transfer(nodeDelegator, amount)) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L194

```
if (!IERC20(asset).transfer(lrtDepositPool, amount)) {
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L86