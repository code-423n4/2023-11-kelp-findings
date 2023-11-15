## \[G‑01\] Save gas by preventing zero amount in mint() and burn()

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

## \[G-02\] Amounts should be checked for 0 before calling a transfer

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

## \[G-03\] Use constants instead of type(uintx).max

Using constants instead of type(uintx).max can save gas in some cases because constants are precomputed at compile-time and can be reused throughout the code, whereas type(uintx).max requires computation at runtime.

```
IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L45

## \[G-04\] use Mappings Instead of Arrays

Mappings, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.

```
address[] public supportedAssetList;
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L19

```
address[] public nodeDelegatorQueue;
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L22

## \[G-05\] The result of a function call should be cached rather than re-calling the function

\* External calls are expensive. Results of external function calls should be cached rather than call them multiple times.

```
function addNewSupportedAsset(address asset, uint256 depositLimit) external onlyRole(LRTConstants.MANAGER) {
        _addNewSupportedAsset(asset, depositLimit);
    }
```

&nbsp;https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L73

```
function setToken(bytes32 tokenKey, address assetAddress) external onlyRole(DEFAULT_ADMIN_ROLE) {
        _setToken(tokenKey, assetAddress);
    }
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L149

## [](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L88)

## \[G-06\] Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry’s script.sol and solmate’s `LibRlp.sol` contracts can help achieve this.

References: <ins>https://book.getfoundry.sh/reference/forge-std/compute-create-address</ins>

<ins>https://twitter.com/transmissions11/status/1518507047943245824</ins>

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
return IStrategy(strategy).userUnderlyingView(address(this));
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L123

## [](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L151C4-L157C6)

## \[G-07\] When possible, use assembly instead of `unchecked{}`

You can also use unchecked{} for even more gas savings but this will not check to see if i overflows. For best gas savings, use inline assembly, however, this limits the functionality you can achieve.

```
unchecked {
                ++i;
            }
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L112C12-L114C14

```
unchecked {
                ++asset_idx;
            }
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L73C13-L75C14

> all loop have this issue

###  \[G -08\]  Compute known value `off-chain`

If You know what data to hash, there is no need to consume more computational power to hash it using keccak256 , you’ll end up consuming 2x amount of gas.

```
bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L21C5-L22C68

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

## \[G-09\] Use assembly to emit an event To efficiently emit events,

it's possible to utilize assembly by making use of scratch space and the free memory pointer. This approach has the advantage of potentially avoiding the costs associated with memory expansion. However, it's important to note that in order to safely optimize this process, it is preferable to cache and restore the free memory pointer.

```
File: blob/main/src/LRTConfig.sol

88:         emit AddedNewSupportedAsset(asset, depositLimit);
103:        emit AssetDepositLimitUpdate(asset, depositLimit);
162:        emit SetToken(key, val);
178:        emit SetContract(key, val);
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L88