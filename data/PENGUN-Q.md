## sETH2 does not have a chainlink price feed.

The new token sETH2 supported by EigenLayer does not have a chainlink price feed and therefore cannot be used in the current implementation.

> Eigenlayer supports stETH, rETH and cbETH as of today. But it intends to add more LSTs in the future through its governance. KelpDao also wants to add support for tokens that Eigenlayer will support in the future.

https://discord.com/channels/810916927919620096/1171865604114882600/1172948255412342824

According to the team, they are trying to support assets that Eigenlayer wants to support. To do so, it is necessary to use a price feed other than chainlink.

## Contract info cannot be changed once it is set.

```solidity
function _setContract(bytes32 key, address val) private {
        UtilLib.checkNonZeroAddress(val);
        if (contractMap[key] == val) {
            revert ValueAlreadyInUse();
        }
        contractMap[key] = val;
        emit SetContract(key, val);
    }

```

contractMap is used to store contract addresses like LRTOracle.
However, once set in _setContract, it cannot be updated.

It is recommended to allow updates for already set keys, so they can be used in situations where contract addresses need to be updated.

## pause does nothing.

LRTOracle has a pause function.

```solidity
/// @dev Triggers stopped state. Contract must not be paused.
    function pause() external onlyLRTManager { //@audit pause nothing
        _pause();
    }

```

However, it does not use modifiers like whenNotPaused that prevent certain actions during a pause, so essentially the pause does nothing.

It is recommended to take measures to prevent major functions like getRSETHPrice from being used during a pause.

## The return value of depositIntoStrategy must be stored.

```solidity
function depositIntoStrategy(
        IStrategy strategy,
        IERC20 token,
        uint256 amount
    ) external onlyWhenNotPaused(PAUSED_DEPOSITS) nonReentrant returns (uint256 shares) {
        shares = _depositIntoStrategy(msg.sender, strategy, token, amount);
    }

```

IEigenStrategyManager.depositIntoStrategy returns the share value for the deposit amount.

It may be useful to record this for future withdrawal features.

Even if there is no withdrawal feature, it is recommended to record this in advance, as it may be difficult to track shares from the time when the withdrawal feature was not available.