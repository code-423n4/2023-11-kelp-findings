# 🛠️ Analysis - Kelp Protocol
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Packages and Dependencies Analysis | Details about the project Packages |
|f) |Other recommendations | What is unique? How are the existing patterns used? |
|g) |New insights and learning from this audit | Things learned from the project |



## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-11-kelp

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-11-kelp#compile)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Kelp](https://blog.kelpdao.xyz/) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/crytic/slither)| The project does not currently have a slither result, a slither control was created from scratch |
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-11-kelp#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-11-kelp#additional-context)||
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|


## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit;
Uses Consensys Solidity Metrics
-  **Lines:** total lines of the source unit
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

![image](https://github.com/code-423n4/2023-11-kelp/assets/104318932/14748089-5d0c-4b17-83c1-54852a332d62)


</br>
</br>

## LRTDepositPool.sol

###   function depositAsset()

![image](https://github.com/code-423n4/2023-11-kelp/assets/104318932/062c69ba-e36b-462b-9fc1-3584b88bf5a8)

## LRTOracle.sol

###   function updatePriceOracleFor()
add/update the price oracle of any supported asset
![image](https://github.com/code-423n4/2023-11-kelp/assets/104318932/6d9c9ad9-137d-430e-925c-32118fbfd275)


</br>
</br>

External Dependencies: The contracts rely on multiple external imports from the OpenZeppelin library, which includes AccessControl, Initializable, ERC20, Pausable, and ReentrancyGuard functionalities​​.

</br>

Known Issues and Context: A significant aspect is the NodeDelegator layer, designed in response to EigenLayer's mechanism to pause deposits. This strategic layer ensures that assets are held and deposited into the EigenLayer asset strategy effectively. All core contracts involved in the Kelp product are upgradeable​​.

</br>

Scope of the Audit: The audit covered 17 contracts with a total of 504 lines of code, utilizing primarily inheritance over composition. It involved a small number of external calls and focused on ERC-20 tokens. A comprehensive understanding of the EigenLayer strategy and Chainlink price oracles was essential for the audit​​.

</br>

Testing and Validation: Extensive testing was conducted, including gas usage reports and compilation of contracts. The testing achieved a high line coverage percentage of 98%, indicating a thorough validation process for the Kelp product's contracts​​.

</br>
</br>

## c) Test analysis
### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content.


### What could they have done better?
- 1) Test suites do not test for re-entrancy
Test teams are testing many functions and variables, but recently, due to the vulnerability in the Vyper Compiler, the hacking of the projects using certain Vyper compiler and losing 50 million $ has revealed the security weakness here. https://cointelegraph.com/news/curve-finance-pools-exploited-over-24-reentrancy-vulnerability

```solidity
nonReentrant  modifier functions : 2 results 

src/LRTDepositPool.sol:
  119:     function depositAsset(
  120:         address asset,
  121:         uint256 depositAmount
  122:     )
  123:         external
  124:         whenNotPaused
  125:         nonReentrant
  126:         onlySupportedAsset(asset)
  127:     {


  183:     function transferAssetToNodeDelegator(
  184:         uint256 ndcIndex,
  185:         address asset,
  186:         uint256 amount
  187:     )
  188:         external
  189:         nonReentrant
  190:         onlyLRTManager
  191:         onlySupportedAsset(asset)
  192:     {

```
The accuracy of the functions has been tested, but it has not been tested whether the `nonReentrant` modifier in the function works correctly or not. For this, a test must be written that tries to reentrancy and was observed to fail.


</br>





## d) Security Approach of the Project

### Successful current security understanding of the project;

1 - First they manage the 1nd audit process with an  Code4rena


### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. https://immunefi.com/





##  e) Packages and Dependencies Analysis 📦

| Package                                                                                                                                     | Version                                                                                                               | Usage in the project                               | Audit Recommendation                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts)         | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts)     |               AccessControlUpgradeable.sol, Initializable.sol, ERC20Upgradeable.sol, ReentrancyGuardUpgradeable.sol, PausableUpgradeable.sol, IAccessControl.sol |-  Version `4.9.3` is used by the project, it is recommended to use the newest version `5.0.0`                                                                                          |



## f) Other recommendations

✅ The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns; https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

✅ It is seen that the latest versions of imported important libraries such as Openzeppelin are not used in the project codes, it should be noted. https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/

✅A good model can be used to systematically assess the risk of the project, for example this modeling is recommended; https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

## g) New insights and learning from this audit

🔎 1. Liquid Restaking Concept: KelpDAO introduces liquid restaking, a DeFi concept that enhances the flexibility and liquidity of staked assets while maintaining their rewards and security. This approach addresses the need for economic security across different protocols and improves capital efficiency for stakers. This was the part that impressed me the most.

🔎 2.  rsETH Development: Under KelpDAO, rsETH is developed as a single liquid restaked token. It can be minted against Liquid Staking Tokens (LSTs) and allows fractional ownership of staked assets, simplifying access to restaking and DeFi opportunities.

🔎 3.  EigenLayer Integration: KelpDAO leverages the EigenLayer protocol, which allows Ethereum stakers to reuse their staked ETH for restaking, thus boosting the security of multiple services simultaneously.

🔎 4.  Addressing Challenges: KelpDAO aims to solve various challenges associated with restaking, such as complex reward structures, high gas fees, liquidity constraints, and the discovery of suitable node operators.

🔎 5.  Future Developments: KelpDAO is planning strategic partnerships, comprehensive security audits, and a mainnet launch. These steps are geared towards enhancing the ecosystem, ensuring security, and bringing new functionalities and opportunities to its users.I liked this part





### Time spent:
14 hours