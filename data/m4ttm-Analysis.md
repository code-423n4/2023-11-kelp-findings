## 1. Audit Approach

|     |     |     |
| --- | --- | --- |
| Step | Task | Details |
| 1   | Run Tests | Tests run successfully |
| 2   | Coverage | 98% test coverage for contracts in audit scope |
| 3   | Slither | Reviewed Slither results, no vulnerabilities discovered |
| 4   | Surya | Generate graphs to understand the overall project structure. Provided an initial insight to the contract inheritance and function call flow |
| 5   | Solidity Metrics | Generate metrics reports to obtain initial insight on the codebase, noting areas of potential concern |
| 6   | Code Review | Line by line code review |
| 7   | Test Review | Review of each test and it's purpose |

## 2. Mechanism Summary

Kelp DAO aims to turn illiquid staked assets on platforms such as EigenLayer into liquid tradable assets through the use of its Liquid Restaked Token, rsETH. Users can stake, restake and keep their funds liquid for further use on other DeFi platforms.

`LRTDepositPool` acts as the entry point for users to deposit their funds and mints `rsETH` which is an ERC20 token used as a receipt representing the underlying assets. This is a tradable liquid token which can be used on further DeFi platforms. The `NodeDelegator` receive these funds and delegates them to an Eigenlayer strategy. `LRTOracle` manages and calls the oracle used by the system to obtain prices. `LRTConfig` is used as a registry for system configuration and a single source of truth for addresses of contracts used in the system.

## 3. Centralisation Risks

### 3.1 Use of Upgradable contracts

The contracts can be upgraded, which poses a risk to users funds should contract logic be changed maliciously allowing an attacker to withdraw users funds. Users should be aware that contracts may change in the future in a way that they are not happy with.

### 3.2 Chainlink Oracle Centralisation

The Chainlink oracle is controlled by a 4 of 8 multisig which is a fairly small group of people.

## 4. Systemic Risks

### 4.1 Use of Chainlink Price Oracle

Chainlink brings off chain price data on chain at specific time intervals. There is no absolute guarantee that this will be correct, as it brought on chain by a third party rather than using on chain code. This may be incorrect should the Chainlink signers collude. There is also the risk of prices being out of date, the heartbeat interval for the desired pairs should be checked on the Chainlink website and the Kelp DAO. team should decide if they consider this appropriate.

### Time spent:
30 hours