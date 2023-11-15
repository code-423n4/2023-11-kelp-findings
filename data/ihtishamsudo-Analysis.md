# [01] Detailed Analysis of Kelp DAO Smart Contracts [In Scope]

## [1.1] General Introduction About Kelp DAO

Here is a general intro to how I looked into this codebase and how I summarize everything I looked for in this codebase. I couldn't find decent vulnerabilities because of lack of time but I understood it enough well to write an analysis.

## [1.2] Brief Introduction About the Codebase In Scope

I looked at the codebase contracts in as sequence as it were actually in and it actually helped me in understanding codebase well.

1. **LRTDeposit.sol**
2. **LRTDepositPool.sol**
3. **LRTOracle.sol**
4. **NodeDelegator.sol**
5. **RSETH.sol**

So, as per the protocol says, These contracts collectively establish the infrastructure for asset management, liquidity provision, oracle-based pricing, and token representation within the Kelp DAO ecosystem.

In the Analysis section I will try to provide Contract By Contract Some Key features, Security Considerations and Some Recommendations.
## [1.3] Analysis of the Contracts in Scope

### [1.3.1] LRTDeposit.sol

#### Key Features:
- Handles LST asset deposits.
- Manages a queue of node delegator contracts.
- Mints rsETH based on deposited assets.

#### Security Considerations:
- Implements role-based access control.
- Contains checks for valid amounts and limits.

#### Recommendations:
- Ensure proper validation and checks for asset transfers.

### [1.3.2] LRTDepositPool.sol

#### Key Features:
- Manages LST asset deposits.
- Determines the current limit of asset deposits.
- Keeps track of node delegator contracts.

#### Security Considerations:
- Implements Pausable and ReentrancyGuard for emergency scenarios.

#### Recommendations:
- Ensure proper validation of asset limits and deposit amounts.

### [1.3.3] LRTOracle.sol

#### Key Features:
- Calculates exchange rates for assets.
- Provides RSETH/ETH exchange rate.
- Allows addition/update of price oracles.

#### Security Considerations:
- Implements Pausable for emergency situations.

#### Recommendations:
- Ensure secure addition/update of price oracles for supported assets.

### [1.3.4] NodeDelegator.sol

#### Key Features:
- Approves the maximum amount of an asset to the eigen strategy manager.
- Deposits assets into the associated strategy.
- Transfers assets back to the LRT deposit pool.

#### Security Considerations:
- Implements Pausable and ReentrancyGuard for emergency scenarios.

#### Recommendations:
- Ensure proper validation and checks for asset transfers and approvals.

### [1.3.5] RSETH.sol

#### Key Features:
- ERC-20 contract for the rsETH token.
- Allows minting and burning of rsETH.
- Implements Pausable for emergency scenarios.

#### Security Considerations:
- Implements Access Control for minting and burning roles.

#### Recommendations:
- Ensure secure minting and burning processes with proper access control.

## [02] Architectural Improvements

Kelp DAO team has written code in pretty well architecture but there are most important things that it lacks and that is fuzzing and invariant testing. I would like to address to team to write proper fuzz and invariant testing to disclose edge cases if they really hold or not. In recent Raft hack, there was an Edge case of rounding error that was missed by Top tier firms and audit competitions so it must be sure that proper invariant and fuzzing is done on codebase. 
Implement comprehensive fuzzing and invariant testing for the smart contracts, including edge cases. This testing approach will enhance the overall security of the contracts by systematically exploring different inputs and scenarios. Tools such as Echidna or Foundry can be utilized to ensure the robustness of the contracts against unforeseen issues. Regular updates to the test suite should be made to cover new edge cases and potential vulnerabilities.

## [03] Centralization Risk

### General Risks:

1. **Configuration Parameters and Initialization:**
   - Parameters set during deployment could lead to centralization if they create unfair advantages.

2. **Curve Equations and Calculations:**
   - Vulnerabilities or inaccuracies in curve equations could be exploited, impacting fairness.

3. **Constraints on Ratios and Balances:**
   - Checks on ratios and balances could lead to centralization if biased towards certain participants.

## [04] Time Spend

- Day 1: Reading the Docs and Info Provided about the Protocol on C4 Audit Page
- Day 2: Thoroughly and Deeply Studied Code
- Day 3: Tried Understanding Code and Finding Weak Spots
- Day 4: Writing Report

### [4.1] Total Time Spent

10 hours

### Time spent:
10 hours