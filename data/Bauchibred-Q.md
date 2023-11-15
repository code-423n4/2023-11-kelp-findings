# QA Report

## Table of Contents

| Issue ID                                                                                                  | Description                                                                                 |
| --------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| [QA-01](#qa-01-transfers-should-be-done-via-a-low-level-call-if-possible)                                 | Transfers should be done via a low-level call if possible                                   |
| [QA-02](#qa-02-multiple-issues-on-how-external-protocols-are-being-integrated-to-kelp)                    | Multiple issues on how external protocols are being integrated to KELP                      |
| [QA-03](#qa-03-protocol-might-break-for-a-token-with-a-proxy-and-implementation-contract-like-cbeth-reth) | Protocol might break for a token with a proxy and implementation contract (like cbETH/rETH) |
| [QA-04](#qa-04-setters-should-always-have-equality-checkers)                                              | Setters should always have equality checkers                                                |
| [QA-05](#qa-05-lrtoracle-sol-should-implement-the-whennotpaused-modifier)                                 | `LrtOracle.sol` should implement the `whenNotPaused()` modifier                             |
| [QA-06](#qa-06-lrtconfiginitialize-could-be-made-more-effective)                                          | `LRTConfig::initialize()` could be made more effective                                      |

## QA-01 Transfers should be done via a low-level call if possible

### Impact

Some tokens do not follow the normal EIP20 specification, i.e instead of returning false on trnasfer failures they revert, which means that the current interface being used wouldn't work with these tokens

### Proof of Concept

Using this search command: `https://github.com/search?q=repo%3Acode-423n4%2F2023-11-kelp+.transfer&type=code` we can see that there are two instances of IERC20.transfer, one of which is in the [LRTDepositPool.sol contract](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L194) when trying to transfer the assets to the node delegator

Issue is just as explaine in the _Impact Section_ if the

```solidity
    function transferAssetToNodeDelegator(
        uint256 ndcIndex,
        address asset,
        uint256 amount
    )
        external
        nonReentrant
        onlyLRTManager
        onlySupportedAsset(asset)
    {
        address nodeDelegator = nodeDelegatorQueue[ndcIndex];
        if (!IERC20(asset).transfer(nodeDelegator, amount)) {
            revert TokenTransferFailed();
        }
    }
```

Issue with this is that the IERC20.transfer interface complies with the transfer that has a return value, i.:

```solidity
    function transfer(address recipient, uint256 amount) external returns (bool);
```

> NB: I did due diligence to see that this does not affect any of the three LSTs currently accepted but as the lists get wider when EigenLayer also integrates more assets this could be an issue.

### Recommended Mitigation Steps

The fix for this would be to call the function via a low level using it's selector, as a low level call wouldn't revert even if the function doesn't return a bool.

## QA-02 Multiple issues on how external protocols are being integrated to KELP

### Impact

Any of the below listed issues going through massively faults the whole depositing/staking functionality of `rsETH`

### Proof of Concept

Take a look at `getAssetPrice()`, key to note that this eventually gets called for any staking depositing or querying of prices executions

```solidity
    function getAssetPrice(address asset) external view onlySupportedAsset(asset) returns (uint256) {
        return AggregatorInterface(assetPriceFeed[asset]).latestAnswer();
    }

```

Issue with this, is that there are multiple safe checks to apply while querying prices from oracles or in this case chainlink that's not being done, below a few are listed:

- There are no checks for stale prices.
  Which inherently leads to ingestion of wrong prices and essentially minting the wrong amount of rsETH for a specific amount of provided supported asset
- Some assets have more than one price oracle, with one being updated more frequent than the other, leading to it being more safe.
  In this sense even if stale checks are applied, if the feed beiung integrated has a 24 hour stale timeframe and there is another one with a 1hr stale timeframe, it's, obviously better to work with the latter inorder to correctly mint the right amount of rsETH per assets supplied.
- Querying prices is not done in a try/catch format.
  This could break the protocol's staking/depositing implementations, as while getting the current RSETH/ETH exchange rate, the function loops through the supported assets checking for their prices and also the amount of assets present, obviously leading to an issue whenever one of the queries to get the prices reverts.
- No circuit break checkers
  This could lead to an issue where the amount of rsETH being minted is either very undervalued or overvalued, due to the price feed return these border prices even if the price is really outside the min/max circuit breakers.
- Casting of prices from int to uint.
  Chainlink returns prices in `int256` since prices could be negative, but current implementation actually just changes these prices to uint without ensuring that the prices are positive. _would only be fair to note that assets that could have -ve prices are RWA and not crypto assets._

### Recommended Mitigation Steps

After dropping the deprecated function as suggested by the bot report, ensure to correctly integrate external protocols, for e.g all fixes to issues that have been highlighted and more can be seen [here](https://medium.com/cyfrin/chainlink-oracle-defi-attacks-93b6cb6541bf).

> NB: This is just a QA report of how bad everything could get with how chainlink/external protocols is/are currently being integrated, some of the aforementioned that could lead to way more impactful issues in protocol have been separately submitted as H/M issues with deep explanation and their respective fixes.

## QA-03 Protocol might break for a token with a proxy and implementation contract (like cbETH/rETH)

### Impact

Tokens whose code and logic can be changed in future can break the protocol and lock users from redeeming that asset.

### Proof of Concept

For a token like cbETH, which has a proxy and implementation contract, if the implementation behind the proxy is changed, it can introduce features which break the protocol, like adding a fee on transfer, or changing the balance over time like a rebasing token.

### Recommended Mitigation Steps

Seems impossible but try limiting the protocol's dependence on upgradeable tokens as viable assets or clearly document this for users.

## QA-04 Setters should always have equality checkers,

### Impact

Low, info on better code structure

### Proof of Concept

Take a look at `updateAssetDepositLimit()`

```solidity
    function updateAssetDepositLimit(
        address asset,
        uint256 depositLimit
    )
        external
        onlyRole(LRTConstants.MANAGER)
        onlySupportedAsset(asset)
    {
        //@audit
        depositLimitByAsset[asset] = depositLimit;
        emit AssetDepositLimitUpdate(asset, depositLimit);
    }
```

As see the function updates the deposit limit for an asset, but there are no checks that this value is not the value that's been previously stored.

### Recommended Mitigation Steps

As a general rule, setters should always have equality checkers, in this case just like the `updateAssetStrategy()` function from the same contract

## QA-05 LrtOracle.sol should implement the `whenNotPaused()` modifier

### Impact

Assumption has been made that protocol implements the `pause()` & `unpause()` so as to limit some of the functionality to only be accessible whenever the protocol is not paused but that's not been followed in `LrtOracle.sol`

### Proof of Concept

Using this search command: `https://github.com/search?q=repo%3Acode-423n4%2F2023-11-kelp+pause&type=code` we can see that there are multiple instances of the pausing mechanism being used in contracts in scope, multiple contracts use this and implement the `whenNotPaused()` modifier with an exception of `LrtOracle.sol` which seems to be a honest mistake from the devs.

### Recommended Mitigation Steps

Either use the `whenNotPaused()` modifier as I assume was intended or remove the functionality from `LrtOracle.sol`all together.

## QA-06 `LRTConfig::initialize()` could be made more effective

### Impact

Low.. info on better code structure

### Proof of Concept

```solidity
    function initialize(
        address admin,
        address stETH,
        address rETH,
        address cbETH,
        address rsETH_
    )
        external
        initializer
    {
        UtilLib.checkNonZeroAddress(admin);
        //@audit QA no need for these checks, since it gets checked in _setToken(), i.e all three asides rsETH
        UtilLib.checkNonZeroAddress(stETH);
        UtilLib.checkNonZeroAddress(rETH);
        UtilLib.checkNonZeroAddress(cbETH);
        UtilLib.checkNonZeroAddress(rsETH_);

        __AccessControl_init();
        _setToken(LRTConstants.ST_ETH_TOKEN, stETH);
        _setToken(LRTConstants.R_ETH_TOKEN, rETH);
        _setToken(LRTConstants.CB_ETH_TOKEN, cbETH);
        _addNewSupportedAsset(stETH, 100_000 ether);
        _addNewSupportedAsset(rETH, 100_000 ether);
        _addNewSupportedAsset(cbETH, 100_000 ether);

        _grantRole(DEFAULT_ADMIN_ROLE, admin);

        rsETH = rsETH_;
    }
        /// @dev private function to set a token
    /// @param key Token key
    /// @param val Token address
    function _setToken(bytes32 key, address val) private {
        UtilLib.checkNonZeroAddress(val);
        if (tokenMap[key] == val) {
            revert ValueAlxreadyInUse();
        }
        tokenMap[key] = val;
        emit SetToken(key, val);
    }
```

As seen the the `_setToken()` already implements a zero address checker, hence making the zero check for the three tokens that get routed via this kinda redundant.

### Recommended Mitigation Steps

Change `initialize()` to:

```diff
    function initialize(
        address admin,
        address stETH,
        address rETH,
        address cbETH,
        address rsETH_
    )
        external
        initializer
    {
        UtilLib.checkNonZeroAddress(admin);
-        UtilLib.checkNonZeroAddress(stETH);
-        UtilLib.checkNonZeroAddress(rETH);
-        UtilLib.checkNonZeroAddress(cbETH);
        UtilLib.checkNonZeroAddress(rsETH_);

        __AccessControl_init();
        _setToken(LRTConstants.ST_ETH_TOKEN, stETH);
        _setToken(LRTConstants.R_ETH_TOKEN, rETH);
        _setToken(LRTConstants.CB_ETH_TOKEN, cbETH);
        _addNewSupportedAsset(stETH, 100_000 ether);
        _addNewSupportedAsset(rETH, 100_000 ether);
        _addNewSupportedAsset(cbETH, 100_000 ether);

        _grantRole(DEFAULT_ADMIN_ROLE, admin);

        rsETH = rsETH_;
    }
```
