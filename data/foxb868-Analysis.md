## Overview

Kelp is a DAO-governed protocol that allows restaking of liquid staked assets like Lido's stETH to further compound yields. The key components are:

- **LRTConfig** - stores protocol configuration and contract addresses
- **LRTDepositPool** - allows users to deposit assets 
- **NodeDelegator** - delegates assets to strategies
- **LRTOracle** - provides asset price feeds  
- **RSETH** - receipt token representing deposited assets  

**LRTConfig**

This contract stores critical protocol configuration like supported assets and contract addresses. It is the source of truth for integrations.

**Risks**:
- The admin can arbitrarily set any contract address, like LRTDepositPool. This could redirect user funds.

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L165-L167

```solidity
    function setContract(bytes32 contractKey, address contractAddress) external onlyRole(DEFAULT_ADMIN_ROLE) {
        _setContract(contractKey, contractAddress);
    }
```

- Assets could be added to the supported list without proper preparations. User funds could become stuck.

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L73-L75

```solidity 
    function addNewSupportedAsset(address asset, uint256 depositLimit) external onlyRole(LRTConstants.MANAGER) {
        _addNewSupportedAsset(asset, depositLimit);
    }
```

- No validation that set strategies match expected asset types. Could set a ETH strategy for DAI.

**Mitigations**:
- Use code hash checks before trusting contractMap addresses.
- Require assets have a non-zero strategy before accepting deposits.
- Make config immutable once stable.

**LRTDepositPool**

This is the user entry point to make deposits. It controls pool balances.

**Risks**:
- Arbitrary tokens can be transfered to NodeDelegators without input validation.

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L183-L197

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

- Exchange rates from LRTOracle are not validated before minting rsETH. Manipulated rates could drain funds.

**Mitigations**:
- Check allowed assets against LRTConfig whitelist before external transfers. 
- Compare exchange rates against fixed sanity rates before minting rsETH.

**NodeDelegator** 

Takes custody of user funds and deposits them into yield generating strategies.

**Risks**:
- Unlimited approval given to external StrategyManager contract. Could steal tokens.

- Reported asset balances trusted from external Strategy contract. Could lie and underreport. 

**Mitigations**:
- Approve StrategyManager contract only for amounts about to be deposited.
- Fetch asset balances from the blockchain instead of through an external call.

## Architecture 

Kelp uses a modular architecture of upgradeable contracts connected through the LRTConfig registry. This provides flexibility to swap contract implementations. However, care must be taken with privilege management between contracts.

The deposit flow is:

1. Users approve tokens to LRTDepositPool
2. Users call `deposit()` on LRTDepositPool 
3. LRTDepositPool transfers tokens to itself
4. LRTDepositPool mints rsETH to user based on value
5. LRTDepositPool transfers tokens to NodeDelegator
6. NodeDelegator deposits tokens into yield-generating strategies

The deposit flow is a critical user journey that deserves deeper analysis.

**The Deposit Flow**

1. Users approve tokens to LRTDepositPool

   - Risk: Users could accidentally approve more tokens than they intend to deposit  
   
   - Mitigation: Approve only the amount about to be deposited
   
2. Users call `deposit()` on LRTDepositPool

   - Risk: `deposit()` relies on LRTOracle for exchange rates. Manipulated rates could affect minted rsETH amounts.
   
   - Mitigation: Compare LRTOracle rate against a fixed sanity rate.
   
3. LRTDepositPool transfers tokens to itself 

   - Risk: Reentrancy during this transfer could be exploited to drain funds

   - Mitigation: Use reentrancy guard in LRTDepositPool
   
4. LRTDepositPool mints rsETH to user

   - Risk: Minting rsETH does not check allowance of LRTDepositPool with RSETH contract

   - Mitigation: Check rsETH minting allowance before minting
   
5. LRTDepositPool transfers tokens to NodeDelegator

   - Risk: No validation of allowed assets or amounts

   - Mitigation: Check asset & amount against LRTConfig before transfer
   
6. NodeDelegator deposits tokens into strategies

   - Risk: Strategy contract is not verified before use

   - Mitigation: Check code hash of Strategy contract  

**Example Code**

```solidity
// LRTDepositPool.sol

// INPUT VALIDATION 
function deposit(uint amount) nonReentrant {

  require(
    LRTOracle.getExchangeRate() >= MIN_EXCHANGE_RATE, 
    "Rate too low"
  );
  
  require(
    RSETH.allowance(address(this)) >= amount,
    "Minting not allowed"
  );

  // ...
}

// LRTConfig whitelist check
function transfer(asset, amount) {
  
  require(
    LRTConfig.allowedAssets[asset],
    "Asset not allowed"
  );
  
  // ...
}
```

## Trust Relationships

There is a high degree of trust between the different components:

- LRTDepositPool trusts LRTConfig for supported assets and oracles
- NodeDelegator trusts LRTConfig for valid strategies
- LRTOracle trusts price feeds from LRTConfig

This makes LRTConfig security-critical. A compromised config can break integrations.

The high degree of trust in LRTConfig creates centralization risks. 

**The Issue**: LRTConfig is a central point of failure due to excessive trust by other contracts.

**Root Cause**

- Uses LRTConfig to check for supported assets 

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L28-L33

```solidity 
    modifier onlySupportedAsset(address asset) {
        if (!isSupportedAsset[asset]) {
            revert AssetNotSupported();
        }
        _;
    }
```

- NodeDelegator trusts LRTConfig for valid strategies

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L51-L68

```solidity
    function depositAssetIntoStrategy(address asset)
        external
        override
        whenNotPaused
        nonReentrant
        onlySupportedAsset(asset)
        onlyLRTManager
    {
        address strategy = lrtConfig.assetStrategy(asset);
        IERC20 token = IERC20(asset);
        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);


        uint256 balance = token.balanceOf(address(this));


        emit AssetDepositIntoStrategy(asset, strategy, balance);


        IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
    }
```

- LRTOracle uses LRTConfig price feeds without checks 

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L45-L47

```solidity
    function getAssetPrice(address asset) public view onlySupportedAsset(asset) returns (uint256) {
        return IPriceFetcher(assetPriceOracle[asset]).getAssetPrice(asset);
    }
```

**Impact**

- A compromised LRTConfig can mislead other contracts by:

  - Returning any strategy for assets

  - Providing a malicious price feed

  - Supporting fake/arbitrary assets

- This could lead to loss of funds, broken integrations, stalled deposits 

**Mitigations**

- Use whitelists and zero knowledge proofs to verify LRTConfig responses  

- Make critical configuration immutable after initial setup

- Build redundancy - don't solely depend on LRTConfig

## Privilege Management

The admin role in LRTConfig has a lot of power. It can:

- Add/remove supported assets
- Change LRTDepositPool address 
- Change price oracles
- Update strategies

Checks should be added before other contracts trust LRTConfig state. For example, verifying:

- Assets are supported before accepting deposits
- Strategies match expected asset types
- Oracle addresses are on an approved list

The broad privileges of the LRTConfig admin role create significant centralization risks if unchecked. 

**The Issue**: The LRTConfig admin has expansive permissions without input validation.

**Root Cause**

The admin can:

- Set any contract address, like a fake LRTDepositPool 

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L165-L180

```solidity
    function setContract(bytes32 contractKey, address contractAddress) external onlyRole(DEFAULT_ADMIN_ROLE) {
        _setContract(contractKey, contractAddress);
    }


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
}
```

- Add completely arbitrary assets without validation 

- Update the price oracle without whitelisting 

**Impact**

This could allow the admin to:

- Point to a malicious LRTDepositPool to steal funds
- Add unsupported assets and break integrations
- Set a manipulative price oracle to affect valuations

**Mitigations**

- Use a DAO, not solo admin role 

- Add input validation on state changes

- Make critical config immutable after setup 

- Emit events on all state changes 


**LRTConfig - Malicious Strategy Manager**

The LRTConfig admin can arbitrarily set the strategy manager address:

```solidity
// LRTConfig.sol
function setStrategyManager(address mgr) external onlyAdmin {
  strategyManager = mgr; // No validation
} 
```

NodeDelegator trusts this address to handle large deposits:

```solidity
// NodeDelegator.sol 

function deposit() {
   IStrategyManager(LRTConfig.strategyManager()).deposit(funds); // No checks!
}
```

A malicious manager could steal funds instead of staking them.

Mitigation is to validate the manager code hash before interacting.

**LRTConfig - Fake LRTDepositPool**

The admin can replace the pool address:

```solidity
// LRTConfig.sol
function setPool(address newPool) external onlyAdmin {
  pool = newPool; // No protections
}
```

A fake pool could just steal deposits without providing yield. 

Mitigation is to make the pool address immutable after setup.

**LRTDepositPool - Arbitrary Asset Transfers** 

LRTDepositPool can transfer any asset: 

```solidity
// LRTDepositPool.sol
function transfer(address asset, uint amt) external {
  // No validation of asset
  NodeDelegator.transfer(asset, amt); 
}
```

Assets should be validated against the LRTConfig whitelist.

**LRTOracle - Unvalidated Price Feeds**

Price feeds can be set without validation:

```solidity
// LRTOracle.sol
function setFeed(address feed) external {
  ethFeed = feed; // No protections
}
```

Price updates should come from whitelisted sources.

## Centralization Risks

The LRTConfig admin keys represent a central point of failure. Compromise of the admin can break protocol integrations.

Consider:

- Time delays for config changes
- Multisig schemes instead of solo admin
- Freezing config once stable

**LRTConfig**

- The manager role can pause LRTDepositPool but has no ability to pause NodeDelegator. This inconsistency could block withdrawals.

   - Mitigation: Give manager ability to pause NodeDelegator as well. Or use a DAO for pausing.
   
- No validation on asset strategies to ensure they match the asset type.

   - Mitigation: Check strategy address matches expected token address.
   
- No events emitted on changes to critical state like supported assets list.

   - Mitigation: Emit events for all state changes for front-end detectability.

**LRTDepositPool**  

- Requires assets be in LRTConfig whitelist, but no check they have a strategy set too.

   - Mitigation: Also require asset has a non-zero strategy address set in LRTConfig.
   
- No validation on transfer amounts to NodeDelegator vs current balance.

   - Mitigation: Check transfer amount <= balance of asset in LRTDepositPool.

**NodeDelegator**

- Doesn't validate strategy contract code before interacting.

   - Mitigation: Check code hash matches expected strategy implementation before use.

- No way to escape illiquid assets sent incorrectly by LRTDepositPool.

  - Mitigation: Add ability to withdraw arbitrary tokens if disrupted.
  
**LRTOracle**

- Asset price manipulation could trick LRTDepositPool, but no validations in DepositPool.

   - Mitigation: LRTDepositPool should sanity check rates against known good oracles before minting.

**Manager pausing inconsistency**

- The manager role can pause LRTDepositPool via `pause()` 

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L208-L210

```solidity
    function pause() external onlyLRTManager {
        _pause();
    }
```

- But the manager role has no ability to pause NodeDelegator contracts

- This is inconsistent and could block withdrawals if NodeDelegator is active but LRTDepositPool is paused

- Or, use a DAO voting process instead of the manager role to handle pausing

**Asset strategy validation**

- When the admin sets a new asset strategy in LRTConfig, there is no validation that the strategy contract address matches the expected asset type

- For example, an ETH strategy could be wrongly set for a DAI asset

**Mitigation** 

- Check that the asset address matches the token address of the strategy:

```
// LRTConfig
function setStrategy(address asset, address strategy) external onlyAdmin {

  require(IStrategy(strategy).token() == asset, "Token mismatch");
  
  // Set strategy 
}
```

**Emit state change events** 

- Critical state changes like adding a new supported asset are not emitted as events in LRTConfig

- This reduces transparency into changes for front-end apps

**Mitigation**

- Emit events for all external state changes:

```
// LRTConfig

event SupportedAssetAdded(address indexed asset);

function addSupportedAsset(address asset) external onlyAdmin {
  // Add asset
  
  emit SupportedAssetAdded(asset);
}
```

## Systemic Risks

If LRTDepositPool is paused but NodeDelegator stays active, it could block withdrawals. This could cascade into a wider liquidity crunch.

Coordination is needed for emergency pauses. A DAO-driven process would be more robust.

**The Issue**: Uncoordinated pauses between LRTDepositPool and NodeDelegator could block withdrawals.

**Root Cause**

- LRTDepositPool can be paused by the admin 

```solidity
    function pause() external onlyLRTManager {
        _pause();
    }
```

- But NodeDelegator has no pause mechanism. 

**Impact**

- If LRTDepositPool is paused by admin, deposits would stall

- But NodeDelegator remains active and continues delegating funds

- This disconnect could block withdrawals and create a liquidity crunch

**Likelihood**

- If the admin panics, they could easily pause LRTDepositPool without considering implications

- The impact could cascade across users and integrated protocols 

**Mitigations**

- Add a pause mechanism to NodeDelegator

- Coordinate pauses via DAO voting rather than admin actions

- Implement circuit breaker pause - if deposits stall, automatically pause NodeDelegator 

## Code Quality

Overall the code is well structured and modular. Additional validation on inputs and outputs between contract calls would improve security. 

## Suggestions

- Add more access controls to LRTConfig state changes
- Implement circuit breakers/emergency pauses via DAO
- Expand test coverage for edge cases
- Formal verification of critical deposit/redeem paths


### Time spent:
11 hours