# QA Report

## Chainlink price feed `decimals` not checked

The price value returned by a Chainlink price feed will have a different
`decimals` value depending on the price feed used. While currently **most** ETH pairs use
18 decimals and USD pairs use 8 decimals (see the price feeds for
[LINK/ETH](https://etherscan.io/address/0xDC530D9457755926550b59e8ECcdaE7624181557#readContract)
and
[LINK/USD](https://etherscan.io/address/0x2c1d072e956AFFC0D435Cb7AC38EF18d24d9127c#readContract)
for example), there is no guarantee that this will be the case for price feeds deployed
in the future. If the decimals are not checked when querying a price feed, 
incorrect decimals may be assumed which can lead to significant accounting errors. Specifically,
in `LRTDepositPool#getRsETHAmountToMint`, the decimals of `getAssetPrice()` is assumed to be
exactly 18, otherwise the returned value could be far smaller than expected, leading to users
being minted far fewer rsETH tokens than intended.

To access a price feeds decimals, simply call `priceFeed.decimals()`.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L38

## Chainlink's `latestAnswer` can return stale or incorrect results

Chainlink's `latestAnswer` is used here to retrieve price feed data,
however there is insufficient protection against price staleness. Only the
price is returned, and it is possible for an outdated price
to be received. See
[here](https://ethereum.stackexchange.com/questions/133242/how-future-resilient-is-a-chainlink-price-feed/133843#133843)
for reasons why a price feed might stop updating.

Inaccurate price data can lead to functions not working as expected and/or
lost funds. For example, if the price of a supported asset drops suddenly but the
oracle is returning an outdated higher price, users can take advantage of this by
calling `LRTDepositPool#depositAsset` to mint themselves more rsETH than their
transferred assets are worth, stealing value from the protocol.

Use the `updatedAt` value from `latestRoundData` to check if the price is stale:

```diff
    function getAssetPrice(address asset) external view onlySupportedAsset(asset) returns (uint256) {
-       return AggregatorInterface(assetPriceFeed[asset]).latestAnswer();
+       (,int256 answer,,uint256 updatedAt,) = AggregatorInterface(assetPriceFeed[asset]).latestRoundData();
+       require(block.timestamp - updatedAt < MAX_DELAY, "stale price");
+       return answer;
    }
```

https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L38

## Chainlink aggregators return the incorrect price if it drops below `minAnswer`

Chainlink aggregators have a built in circuit breaker if the price of an asset
goes outside of a predetermined price band. The result is that if an asset
experiences a huge drop in value (i.e. LUNA crash) the price of the oracle will
continue to return the `minAnswer` instead of the actual price of the asset. See
[Chainlink's docs](https://docs.chain.link/data-feeds#check-the-latest-answer-against-reasonable-limits)
for more info.

This discrepency could allow users to `deposit` supported assets that the protocol
perceives as worth more than they actually are, minting them more rsETH than intended
and causing bad debt in the protocol. A similar attack occurred to 
[Venus on BSC when LUNA imploded](https://rekt.news/venus-blizz-rekt/).

Consider adding a check to revert if the price received from the oracle is
out of bounds.

https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol#L38

## `DEFAULT_ADMIN_ROLE` can renounce role

`LRTConfig` inherits OpenZeppelin's `AccessControlUpgradeable` which allows callers to renounce roles via the
`renounceRole` function. This includes the `DEFAULT_ADMIN_ROLE`, which is
the admin for every other role, as well as the role responsible for many critical functions
in the protocol, including `updateAssetStrategy`, `unpause`, etc. If this role is renounced, then calling
any of those functions, as well as management of
all other roles is impossible.

Renouncing this role should be explicitly prevented by overriding `renounceRole` and throwing
if `DEFAULT_ADMIN_ROLE` is passed.

## `LRTConfig#setToken` should require the asset to be supported

If `setToken()` is called on an `assetAddress` that is not supported, it may mislead those who would call `getLSTToken()` and receive a non-zero address.

```solidity
File: src\LRTConfig.sol

149:     function setToken(bytes32 tokenKey, address assetAddress) external onlyRole(DEFAULT_ADMIN_ROLE) {
150:         _setToken(tokenKey, assetAddress); // @audit no check for isSupportedAsset[assetAddress]
151:     }
152: 
153:     /// @dev private function to set a token
154:     /// @param key Token key
155:     /// @param val Token address
156:     function _setToken(bytes32 key, address val) private {
157:         UtilLib.checkNonZeroAddress(val);
158:         if (tokenMap[key] == val) {
159:             revert ValueAlreadyInUse();
160:         }
161:         tokenMap[key] = val;
162:         emit SetToken(key, val);
163:     }
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L149-L163

## LRTManager can transfer asset tokens to `NodeDelegator` while contract is paused

`LRTDepositPool` uses `Pausable` to allow the manager/admin to pause the contract and prevent any sensitive actions, however `transferAssetToNodeDelegator()` is missing the `whenNotPaused` modifier, meaning it is callable even if the contract is in a paused state. The transfer of funds should not be permitted while the contract is paused (in `NodeDelegator`, functions that transfer funds use the `whenNotPaused` modifier even if they are admin protected).

Add the `whenNotPaused` modifier to `transferAssetToNodeDelegator()`.

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L183-L192

## `LRTConfig#updateAssetDepositLimit()` can cause DoS

`LRTConfig#updateAssetDepositLimit()` is used by the LTR manager to change the `depositLimit` for a particular asset after initialization.

```solidity
File: src\LRTConfig.sol

094:     function updateAssetDepositLimit(
095:         address asset,
096:         uint256 depositLimit
097:     )
098:         external
099:         onlyRole(LRTConstants.MANAGER)
100:         onlySupportedAsset(asset)
101:     {
102:         depositLimitByAsset[asset] = depositLimit;
103:         emit AssetDepositLimitUpdate(asset, depositLimit);
104:     }
```
https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L94-L104

However it is lacking checks on the new `depositLimit` value, meaning it is possible to set it below the amount of currently deposited assets, and doing so would result in DoS. This could occur by mistake, or intentionally by a malicious/compromised manager. 

If the `depositLimit` is set to a value less than the current total assets deposited, then `getAssetCurrentLimit()` would revert due to underflow:

```solidity
File: src\LRTDepositPool.sol

56:     function getAssetCurrentLimit(address asset) public view override returns (uint256) {
57:         return lrtConfig.depositLimitByAsset(asset) - getTotalAssetDeposits(asset); // @audit underflow
58:     }
```
https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L56-L58

`getAssetCurrentLimit()` is called within `depositAsset()`, meaning that in this scenario users would not be able to deposit. Consider preventing this by requiring that the new `depositLimit` passed to `updateAssetDepositLimit` is less than `getTotalAssetDeposits()`.

## Ensure `updateAssetStrategy()` cannot set invalid strategy address

A malicious/compromised manager (or one who makes a mistake) could set an invalid strategy address, which could potentially lead to a large loss of user funds. Prevent this by checking that the destination address is indeed an EigenLayer strategy. This can be done by calling `explanation()` on the target address and checking it is equal to the expected result (it is the same for each of the three strategies [[1]](https://etherscan.io/address/0x54945180dB7943c0ed0FEE7EdaB2Bd24620256bc#readProxyContract) [[2]](https://etherscan.io/address/0x93c4b944D05dfe6df7645A86cd2206016c51564D#readProxyContract) [[3]](https://etherscan.io/address/0x1BeE69b7dFFfA4E2d53C2a2Df135C388AD25dCD2#readProxyContract)).

```diff
    function updateAssetStrategy(
        address asset,
        address strategy
    )
        external
        onlyRole(DEFAULT_ADMIN_ROLE)
        onlySupportedAsset(asset)
    {
        UtilLib.checkNonZeroAddress(strategy);
+       require(IStrategy(strategy).explanation() == "Base Strategy implementation to inherit from for more complex implementations");
        if (assetStrategy[asset] == strategy) {
            revert ValueAlreadyInUse();
        }
        assetStrategy[asset] = strategy;
    }
```