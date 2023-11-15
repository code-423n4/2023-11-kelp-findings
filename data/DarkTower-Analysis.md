# KelpDAO Protocol Analysis Report

## Introduction
This analysis report goes over the various components and sections of the KelpDAO Protocol as it has been audited by the DarkTower team. During the duration of this audit, we aimed primarily to gain a comprehensive understanding of the codebase and try to figure out issues that may become vulnerable to the protocol, it's functioning and ultimately affect its users.

#### Findings Summary
Severity  | Instances 
------------- | -------------
Medium  | 1
Low     | 3

## Comments for judge to contextualize the findings
This is to help the judge reduce the time spent on judging. Our findings are mainly chainlink implementations issues that protocol team need to fix before going live. For this audit, we are not submitting low/gas or non critical issues.


## Approach taken in evaluating the codebase
* We began the audit on the day of its audit directly going over the protocol's documentation to get an idea of components and architecture of the protocol.

* We moved to codebase first looking into KelpDAO Chainlink implementations contracts and then process went on. We have confirmed the functionality and understand the contracts well enough and with our auditing skills, have analyzed the contracts with different attack possibilities.

* At first we have seen some mistake in implementation of Chainlink price oracle contract and found some medium vulnerabilities the protocol should look into fixing with our suggestions or other best practices they put together in that regard. 

* We conducted the tests provided by the protocol team and tried to figure out some serious edge cases covered and not yet covered with the tests.

* Finally we wrote the reports and submitted to the c4 portal.


## Codebase Overview
Kelp is a collective DAO designed to unlock liquidity, DeFi and higher rewards for restaked assets. Kelp allows ethereum users to restake their stacked ethereum to get rewarded for those staking amount in protocol such as EigenLayer. 


## Architecture Recommendation
## Current Architecture 
There are currently 4 components of the protocol:
1. Oracle: The Chainlink oracle is currently used in the protocol fetch the price feed of liquidity staking tokens.
2. Node Delegators: The funds from the deposit pool goes to these contracts and these contracts help to provide liquidity to different strategies including EigenLayer. 
3. Deposit Pool: The pool where funds received from the users are accumulated. The funds goes to Node Delegators from this contract.
4. RSETH: The token users get on depositing staked eth to the protocol.

## Recommendations
1. There is no way users can withdraw their liquidity, we suggest the protocol to implement functionality to withdraw their liquidity with a mechanism.
2. Functionality of the protocol such as `_pause` can be called by only managers but we suggest to implement such that admins can also call this function to pause.
3. As price calculation is dependent on Chainlink price oracle, we suggest that the protocol implements the latest functions of Chainlink price oracle suggested by Chainlink.
## Codebase Quality Analysis
Overall, the quality of the codebase is excellent. The covered tests are looking good. 
Category  | Comments | Results
------------- | ------------- | -------------
Code Comments  | NatSpc followed but missing in some places | Excellent
Documentation | Not providing proper documentations of the functionalities and contracts | Not Good
Complexity Management    | Protocol handles the complexity of the codebase properly | Excellent
Other test coverage    | Most of the functional tests are covered with Foundry  | Excellent

## Centralization Risks
1. In current implementation, users can't directly deposit into Node Delegators or Strategies. If the LRTAdmin doesn't transfer the funds in time, the funds might struck in `DepositPool` and `NodeDelegators` contract as there is no way user can remove the funds from the protocol.

## System Risks
1. Using a oracle with less care might results into loss of funds for users if price feed isn't sanitized properly.
2. The rewards mechanism is dependent on EigenLayer strategy. If something goes incorrectly in EigenLayer side, user might face losses of funds. 

## Time Spent
5 hours.










### Time spent:
5 hours