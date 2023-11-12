## Titlte: Suboptimal Inheritance in `ChainlinkPriceOracle`.

## Impact

The `ChainlinkPriceOracle` contract inherits from `IPriceFetcher` instead of directly implementing the `ILRTOracle` interface. This is confusing and doesn't fully capture the intent that this is an oracle implementation.

## Proof of Concept

**[ChainlinkPriceOracle.sol](https://github.com/code-423n4/2023-11-kelp/blob/4b34abc952205e2a34bff893a0de0c75b8052149/src/oracles/ChainlinkPriceOracle.sol#L15-L19)**

```solidity
/// @title ChainlinkPriceOracle Contract 
/// @notice contract that fetches the exchange rate of assets from chainlink price feeds
contract ChainlinkPriceOracle is IPriceFetcher, LRTConfigRoleChecker, Initializable {

  // ...

}
```

This shows `ChainlinkPriceOracle` inheriting from `IPriceFetcher` instead of `ILRTOracle`.

The `ChainlinkPriceOracle` contract currently inherits from:

- `IPriceFetcher` 
- `LRTConfigRoleChecker`
- `Initializable`

But since this is a specific implementation of the `ILRTOracle` interface, it would be clearer to do:

```solidity
contract ChainlinkPriceOracle is ILRTOracle, LRTConfigRoleChecker, Initializable {

  // ...

}
```

The benefits:

- Makes the intent more clear - this implements the oracle interface

- Avoids confusing naming with `IPriceFetcher`

- Allows directly implementing `ILRTOracle` methods

So in summary, changing the inheritance to directly implement the oracle interface better captures the intent and simplifies the contract API.

## Recommended Mitigation

Change the inheritance to:

```solidity 
contract ChainlinkPriceOracle is ILRTOracle, LRTConfigRoleChecker, Initializable {

  // ...

}
```

Directly inheriting from the oracle interface better conveys that this is an implementation contract.


## Title: Unnecessary Max Token Approval in NodeDelegator.

## Impact

The `maxApproveToEigenStrategyManager` function approves the EigenLayer manager contract to spend max uint256 tokens from the NodeDelegator. However, `depositAssetIntoStrategy` then only deposits a specific `balance` amount.

This max approval is unnecessary and increases the damage if the EigenLayer manager contract is compromised.

## Proof of Concept

`maxApproveToEigenStrategyManager` approves max tokens: https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/NodeDelegator.sol#L35-L68

The `depositAssetIntoStrategy` function in NodeDelegator approving the maximum token amount to the EigenLayer manager does seem unnecessary.

Specifically, this line: [L45](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/NodeDelegator.sol#L45)

```solidity
IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);
```

Is approving the EigenLayer manager contract to spend up to 2^256 - 1 tokens.

However, it seems this large approval is not needed, since `depositAssetIntoStrategy` then deposits a specific `balance` amount: [L67](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/NodeDelegator.sol#L67)

```solidity
IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(
  IStrategy(strategy), 
  token,
  balance
);
```

So it would be better to approve only the `balance` amount that is actually being deposited:

```solidity
IERC20(asset).approve(eigenlayerStrategyManagerAddress, balance); 
```

The risk of approving max is it would allow the EigenLayer manager to potentially drain all tokens if compromised, rather than only the deposit amount.

So approving the specific deposit amount is likely sufficient and reduces unnecessary over-approval.

## Recommended Mitigation Steps

1. In `maxApproveToEigenStrategyManager`, approve only the `balance` amount instead of max tokens:

```solidity

IERC20(asset).approve(eigenlayerStrategyManagerAddress, balance);

```

2. Consider removing the separate approval function and approving `balance` inline in `depositAssetIntoStrategy`.

This reduces the excess token approval and limits damages in case of an EigenLayer manager compromise.

## Title: Break admin role into separate admin and governance roles for better separation.

Currently, the `DEFAULT_ADMIN_ROLE` in `LRTConfig` has full privileges including:

- Adding/removing supported assets
- Updating deposit limits
- Pausing contracts
- Upgrading contracts
- Changing admin accounts
- Managing roles

So it has a mix of administrative, governance and upgrade related responsibilities.

My suggestion is to split the `DEFAULT_ADMIN_ROLE` into two separate roles:

1. **Admin Role**: This will be responsible for urgent admin functions like pausing, upgrades, and changing admin accounts.

2. **Governance Role**: This will be responsible for governance related activities like adding assets, changing parameters, risk management etc.

Some benefits of splitting the single admin role:

- Clear separation between urgent admin tasks and governance activities.

- Different account/parties can manage each role.

- Admin role has minimum privileges needed for emergency scenarios.

- Governance party cannot perform sensitive tasks like upgrading contracts or changing admin accounts. 

- Governance activities like adding assets require social consensus so may need broader oversight.

To implement this, the `AccessControl` roles can be split as:

```solidity
// Admin role
bytes32 public constant ADMIN_ROLE = keccak256("ADMIN_ROLE");

// Governance role 
bytes32 public constant GOVERNANCE_ROLE = keccak256("GOVERNANCE_ROLE");
```

The privileges would be divided between the two roles.

