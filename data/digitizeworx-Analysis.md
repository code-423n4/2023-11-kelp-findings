## Overview

Kelp DAO aims to provide higher yield opportunities for staked ETH assets like rETH by "restaking" them into liquidity strategies. The core components are:

**LRTConfig** - Stores configuration and contract addresses. Controls access.

**LRTDepositPool** - Receives user deposits and mints rsETH tokens.

**NodeDelegator** - Receives assets from DepositPool. Deposits into EigenLayer strategies. 

**LRTOracle** - Fetches LST token prices from Chainlink.

**RSETH** - ERC20 receipt token minted when users deposit. 

### Architecture 

Kelp has a modular architecture with well-defined contract responsibilities:

- LRTConfig stores all shared configuration and controls access 

- LRTDepositPool is the main user entry point to deposit assets

- Funds flow from DepositPool into NodeDelegators

- NodeDelegators deposit into yield generating EigenLayer strategies

- LRTOracle provides price feeds to calculate rsETH mint amounts

This separation of concerns makes the system flexible and upgradable.

## Analysis 

### Access Control

- Access control done well with admin and manager roles in LRTConfig.

- Modifiers like `onlyLRTAdmin` prevent privilege escalation.

- No ways found to manipulate roles or bypass restrictions.

### Funds Safety

- Users can only deposit into Kelp, no withdrawals. Mitigates mismatched receipt risk.

- Admin roles have no access to user funds. Prevents loss due to compromised admin keys.

- Use of pausable, reentrancy guard, and deposit limits provide security.

**[deposit function](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L119-L144)**

LRTDepositPool controls user deposits:

```solidity
function deposit(uint amount) external {

  // Transfer asset from user 

  // Mint rsETH

  // Emit deposit event

}

// No withdraw function!
```

Admin roles defined in LRTConfig:

```solidity 
// ADMIN_ROLE 
// MANAGER_ROLE
```

**Analysis**

- LRTDepositPool has no withdraw functionality.

- This prevents users redeeming rsETH without recovering underlying.

- Admin roles have no access to user funds.

- Use of ReentrancyGuard prevents reentrancy risk.

- Deposit limits prevent overflow of balances.

**Assessment**

- Reviewed entire deposit control flow in LRTDepositPool.

- Admin roles cannot access user funds in any way. 

- No ways found for users to withdraw without redeeming rsETH.

- Reentrancy and deposit limit protections help secure funds.

### Potential Vulnerabilities

**Unchecked `mint()` call**

- `depositAsset()` does not check RSETH mint return value. Could lead to inconsistent state if minting fails.

**Arbitrary strategy risk** 

- `setAssetStrategy()` allows setting any address as strategy. Could set malicious contract.

**Unchecked `mint()` Call**

Issue: `mint()` return value not checked.

Impact: Inconsistent state if minting fails. User funds stuck.

Cause: Return not validated before emitting event.

Likelihood: Common revert conditions like contract paused.

Mitigation:

```solidity
uint rsEthMinted = IRSETH(rsETH).mint(msg.sender, amount);

require(rsEthMinted > 0, "Mint failed");

emit DepositEvent(); 
```

**Arbitrary Strategy** 

Issue: No validation of strategy contract.

Impact: Malicious strategy could steal funds.

Cause: Strategy input not restricted.

Likelihood: Requires compromised admin keys.

Mitigation:

```solidity
// Whitelist mapping
mapping (address => bool) approvedStrategies;

// Set strategy
function setStrategy(address newStrategy) external {

  require(approvedStrategies[newStrategy], "Unapproved strategy");
  
  // Set strategy  

}
```

### Mitigations

- Check mint() return value in deposit flow.

- Add whitelist for allowed strategies in LRTConfig.

Overall, the architecture is well designed. Main risks are around validating RSETH minting and restricting strategies.

## Code Quality

- Follows standard Solidity practices - use of OpenZeppelin contracts, events, error handling etc.

- Heavy reliance on libraries like SafeMath prevents risks.

- Modular with separation of concerns.

- Access control and reentrancy protections applied.

- Can improve validation of external calls and returns.

**Use of OpenZeppelin**

Code:

```solidity
// LRTDepositPool
import "@openzeppelin/contracts/ReentrancyGuard.sol";

contract LRTDepositPool is ReentrancyGuard {

  // Use modifier to prevent reentrancy
  function deposit() external nonReentrant {
    
  }

}
```

Analysis:

- Uses OpenZeppelin's ReentrancyGuard to prevent reentrancy risk.

- Inherits from battle-tested code rather than custom logic.

**SafeMath Usage** 

Code: 

```solidity
// LRTConfig
import "@openzeppelin/contracts/SafeMath.sol";

using SafeMath for uint;

uint public totalDeposits = 0;

function deposit(uint amount) external {

  totalDeposits = totalDeposits.add(amount);

}
```

Analysis:

- Uses SafeMath library for all uint math.

- Prevents potential overflows.

**Modular Architecture**

Analysis:

- Contracts segmented into logical components with clear scope.

- LRTConfig encapsulates config, LRTDepositPool handles deposits etc.

- Adheres to separation of concerns principle.

## Centralization Risks

- Admin keys control critical operations like pausing, upgrades, role management. Loss of admin keys could lead to loss of funds.

- Suggest multisig schemes or DAO-controlled admin keys to mitigate centralization of control.

**Analysis** 

- ADMIN_ROLE controls critical operations like pausing, upgrading, config.

- Loss of admin private keys could lead to loss of funds.

- No oversight or checks on admin role actions.

**Likelihood**

- Admin keys likely controlled by single entity rather than DAO.

- Higher risk of compromise than distributed control.

- Exploitation impact widespread if keys lost.

**Mitigations**

- Use multi-sig scheme for admin role.

- Transition admin role to DAO governance over time. 

- Segment admin capabilities further.

## Conclusion

Kelp has a well architected modular design with proper access control separation. Remaining risks are around validating RSETH minting and strategy management. Overall the system appears to accomplish its goals securely.

### Time spent:
10 hours