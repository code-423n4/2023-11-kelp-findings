## [L-01] Missing input validation in `_setToken()`, token addresses in mapping should be immutable.
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L156

```solidity
    mapping(bytes32 tokenKey => address tokenAddress) public tokenMap;
    .
    .
    .
    /// @dev private function to set a token
    /// @param key Token key
    /// @param val Token address
    function _setToken(bytes32 key, address val) private {
        UtilLib.checkNonZeroAddress(val);
        if (tokenMap[key] == val) {
            revert ValueAlreadyInUse();
        }
        tokenMap[key] = val;
        emit SetToken(key, val);
    }
```

The function checks if the new address `val` is already set to that value in the mapping: `if(tokenMap[key] == val)`.
However this is not enough. Token addresses will not change, so values in the mapping should be immutable.

This error can occur if an admin accidentally sets the address for a future token to an existing one in the mapping like `LRTConstants.ST_ETH_TOKEN` instead of the actual token address, for example `LRTConstants.NEW_ETH_TOKEN`, constant names are easy to mistake for one another. This unwanted action can result in sending the wrong type of tokens to a contract.

Recommendation is when setting the token also check if the `tokenMap[key]` is already set to a non-zero address, if it is then revert.

## [L-02] Missing input validation in `_setContract()`, contract addresses in mapping should be immutable.
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L172

```solidity
    mapping(bytes32 contractKey => address contractAddress) public contractMap;
    .
    .
    .
    /// @dev private function to set a contract
    /// @param key Contract key
    /// @param val Contract address
    function _setContract(bytes32 key, address val) private {
        UtilLib.checkNonZeroAddress(val);
        if (contractMap[key] == val) {
            revert ValueAlreadyInUse();
        }
        contractMap[key] = val;
        emit SetContract(key, val);
    }
```

The function checks if the new address `val` is already set to that value in the mapping: `if(contractMap[key] == val)`.
However this is not enough. Contract addresses will not change since proxies are used for upgrading, so values in the mapping should be immutable.

This error can occur if an admin accidentally sets the address for a contract to a key in the mapping that is already set to a different contract address, for example setting the address of `LRT_ORACLE` to the address of `LRT_DEPOSIT_POOL`. This unwanted action will have serious consequences, like sending tokens to the wrong contract and locking them and basically breaking most functionality of the protocol.

Reccomendation is when setting the contract also check if the `contractMap[key]` is already set to a non-zero address, if it is then revert.

## [L-03] No way to remove potential malicios token in `LRTConfig`.
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L109

```solidity

    address[] public supportedAssetList; 
    .
    .
    .
    /// @dev private function to add a new supported asset
    /// @param asset Asset address
    /// @param depositLimit Deposit limit for the asset
    function _addNewSupportedAsset(address asset, uint256 depositLimit) private {
        UtilLib.checkNonZeroAddress(asset);
        if (isSupportedAsset[asset]) {
            revert AssetAlreadySupported();
        }
        isSupportedAsset[asset] = true;
        supportedAssetList.push(asset);
        depositLimitByAsset[asset] = depositLimit;
        emit AddedNewSupportedAsset(asset, depositLimit);
    }
```
An issue about this array being only able to grow was reported as low in the automated findings about, but this is a about a different attack vector.

If we look at the consensys diligece audit of EigenLayer https://consensys.io/diligence/audits/2023/03/eigenlabs-eigenlayer/#potential-reentrancy-into-strategies There is potential reentrancy vulnerability in StrategyBase, particularly related to malicios tokens. StrategyBase is the base contract for strategies, which will be utilized by Kelp. EigenLayer is a third party protocol, so there is a dependency on it's security in the first place. Additionally, Kelp and EigenLayer will utilize other parties tokens, which if they are upgradeable is a vulnerability on it's own. This is a potential risk if the tokens are not properly vetted before approval.

Recommendation is to add a way to remove a particular token from supported asset list or pause certain actions with it during an emergency as a way to mitigate the results of such a scenario.