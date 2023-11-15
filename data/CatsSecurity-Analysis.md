# Kelp Protocol Analysis

## Description of Kelp
Kelp represents a collaborative DAO with the purpose of enhancing liquidity, DeFi, and maximizing rewards for assets that are restaked. The core of this initiative lies in the integration of a single liquid restaked token made for recognized LSTs, ensuring seamless accessibility. The protocol will allow users to employ their rsETH within EigenLayer strategies, enabling them to generate returns. The ongoing development of rsETH will be carried out within the framework of the Kelp brand.
## 1. System Overview

### 1.1 Scoping

- [LRTConfig](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol): This contract includes the functionality to configure the system such as adding new supported assets, updating assets' deposit limit, updating an asset's strategy, setting the `rsETH` contract address, setting the token address and numerous getter functions.
- [LRTDepositPool](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol): This contract is the main user-facing contract and includes the functionality to deposit assets, mint `rsETH`, add node delegator's to the queue, transfer assets to node delegators, update the max node delegators count and numerous getter functions.
- [NodeDelegator](https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol): This contract is the recipient of funds from the [LRTDepositPool](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol) contract and in turn sends them to the EigenLayer strategies, includes functionality to approve assets to EigenLayer, deposit assets into the strategy, transfer funds back to [LRTDepositPool](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol), and numerous getter functions.
- [LRTOracle](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol): This contract is utilized as the `Oracle` to get prices of liquid staking tokens, includes functionality to update the price oracle for supported assets, as well as fetch supported asset and `rsETH` prices.
- [RSETH](https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol): This contract is the `rsETH` ERC20 Token the protocol uses and contains all the standard functionality as per OpenZeppelin's requirements.
- [ChainlinkPriceOracle](https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol): This contract is the `Chainlink Price Oracle` wrapper to integrate their oracles in the [LRTOracle](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol) and contains all standard Chainlink Oracle functionality. 
- [LRTConfigRoleChecker](https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConfigRoleChecker.sol): This contract is abstract and is responsible for handling role checks such as `onlyLRTManager`, `onlyLRTAdmin`, `onlySupportedAsset`, and can also be used to update the [LRTConfig](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol) contract.
- [UtilLib](https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/UtilLib.sol): This contract is a simple helper library that contains a check for non zero address function.
- [LRTConstants](https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConstants.sol): This contract is contains all the shared constant variables used within the system such as `R_ETH_TOKEN`, `ST_ETH_TOKEN`, `CB_ETH_TOKEN`, `LRT_ORACLE`, `LRT_DEPOSIT_POOL`, `EIGEN_STRATEGY_MANAGER`, `MANAGER`.

### 1.2 Access Control Roles
The contracts uses the following 4 access control modifiers for different mechanisms within the system: 

- `DEFAULT_ADMIN_ROLE`: Used for core administrative functions within the system that have to do with initial setting of addresses as well as updating asset strategies. 
- `onlyLRTADMIN`: Used for functionality within the system that has to do with node delegaotrs and unpausing the contracts.
- `onlyLRTMANAGER`: Used for functionality within the system that has to do with node delegators, transfer of funds between contracts and EigenLayer strategies and price oracle configurations.
- `LRTConstants.MANAGER`: Used for functionality within the system that has to do with supported assets and their deposit limits.

### 1.3 Access Restricted Functions

- `DEFAULT_ADMIN_ROLE`: This role has access to administrative functions such as [updating asset strategies](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L109), [setting the rsETH address](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L144), [setting the token address](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L149) & [setting the contract address](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L165).
- `onlyLRTADMIN`: This role has access to functions related to the Node Delegators such as [adding new node delegator contracts to the queue](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L162), [updating the maximum node delegator count](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L202) and [unpausing](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L213) the various contracts that allow for that.
- `onlyLRTMANAGER`: This role has access to other functions related to the Node Delegators such as [transferring assets from the deposit pool to the node delegator contracts](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L183), as well as [approving assets to the EigenLayer strategy manager](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L38), [depositing assets from node delegators to the strategy](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L51), [transferring funds from node delegators back to the deposit pool](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L74), [updating the price oracle of suported assets](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L88), [adding or updating the oracle price feed for assets](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/oracles/ChainlinkPriceOracle.sol#L45), and [pausing](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L208) the various contracts that allow for that.
- `LRTConstants.MANAGER`: This role has access to functions related to [adding new supported assets](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L73), and [updating the asset deposit limits](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L94).

## 2. Architecture Overview & Diagrams

### 2.1 Architecture Overview

- LRTConfig: The LRTConfig contract is a critical part of the Kelp protocol. It contains the logic for the system's configuration such as supported assets and their deposit limits and asset strategies.

- LRTDepositPool: The LRTDepositPool contract is the main user-facing contract in the protocol responsible for taking in deposits and minting `rsETH` for users. Users can also get information related to current asset limits, total asset deposits and the node delegator queue. Restricted roles have access to node delegator and transferring of funds functions inside this contract.

- LRTOracle: The LRTOracle contract is the one responsible for fetching prices of LSTs (liquid staking tokens). It works together with the LRTDepositPool to maintain accurate price validity and helps in calculating how much `rsETH` to mint for users.

- RSETH: The RSETH Contract is the system's ERC20 token implementation. Not much to say here.

- ChainlinkPriceOracle: The ChainLinkPriceOracle contract is the wrapper contract used to integrate Chainlink's Oracles. It works together with the LRTOracle contract to supply it's fetched data.

- LRTConfigRoleChecker: The LRTConfigRoleChecker contract is the system's implementation for access control modifiers enabling certain functions to only be accessible by privileged roles.

- UtilLib: The UtilLib contract is a helper function lib that contains a check for non zero address function.

- LRTConstants: The LRTConstants contract contains the constant variables shared between the contracts such as `R_ETH_TOKEN` & other supported LSTs, `LRT_ORACLE` & other contracts, as well as the `MANAGER` role.

### 2.2 Architecture Diagram
The architecture diagram visualizes how the contracts in the system interact with eachother and provides short descriptions of the contracts' intended mechanism.

![Architechture - Frame 1](https://user-images.githubusercontent.com/58657666/282805871-37965b8f-cece-4a75-845a-0437ae75d183.jpg)

### 2.3 Flow of Funds Diagram
The flow of funds diagram visualizes the process through which a user goes in order to deposit their assets into the protocol and how they travel to their final destination of the EigenLayer strategies.

![Architechture - Frame 2](https://user-images.githubusercontent.com/58657666/282804967-c299fa9e-ff8b-43ed-9c91-efaee8b950b2.jpg)

### 2.4 Core Contracts Parameters Diagram
The core contracts parameters diagram visualizes the main parameters and/or storage variables in the system and what they represent.

![Architechture - Frame 3](https://user-images.githubusercontent.com/58657666/282804985-134b842b-1bce-4991-981f-83f15c9c7f21.jpg)

## 3. Codebase Assessment
The total nSLOC of the contracts in scope are quite low but it can be seen that the developers have done an excellent job of integrating all the functionality properly. Config, Deposit Pool, Oracles & Node Delegaotrs communicate seamlessly with access control roles functioning as intended wherever needed. The system includes `pause` functionality which, in my opinion, is always a good idea.

- General Redability: The codebase exhibits sound coding practices, upholding readability and organization through well-defined variables and function names. 
- Code Style: The code adheres to a uniform style and indentation pattern, complemented by in-line comments that provide comprehensive guidance throughout it's mechanisms.
- Gas: Gas efficiency is effectively upheld.
- Test Suite: With a test suite coverage of 98%, the protocol did an almost perfect job. The setting up of environments for tests was easy and straight-forward.
- Errors & Events: This is something that codebase lacked in. I didn't come across any handling neither of errors, nor events.
- Upgradability: All core contracts of the protocol maintained upgradabillity, with plans of future expansion & integration this should be a must.
## 4. Level of Documentation
The documentation provided in the github repo was not extensive, although it's understandable for such a small codebase. There was a link to [their blog](https://blog.kelpdao.xyz/reimagine-restaking-with-rseth-developed-by-kelp-78b7a5ae1230) with some additional information on more in-general subjects such as "What is restaking", "What is EigenLayer" and such. That's all good but I would have liked more techinical documentation about the project.
## 5. Systemic & Centralization Risks

### 5.1 Systemic Risks
Although the codebase is small, since this is the protocol's first audit I wouldn't be surprised if a few bugs arise, it is our goal to clear them. I would usually say there is always room for improvement codebase-wise but I belive the developers have done an excellent job here. The owner's have stated that they intend to expand the codebase and conduct subsequent audits for the expansion which is great. Me and my teammate were able to uncover a few vulnerabilities and I am sure other auditors will come up with more stuff.

- Loss of value upon mint: Our team uncovered a vulnerability which if exploited could set the system up in such a way that the 2nd depositor's minting value is significantly reduced. This is quite has quite a serious impact although potentially only possible for the 2nd depositor, even if the system currently has no `withdraw` functionality, it is still a major loss of `value`. The issue arises from the fact that the codebase
 does not use internal accounting during a crucial division to calculate the `rsETH` amount to mint, but instead takes the contracts' balances. This opens the door for an attacker to simply donate/directly send value instead of going through the `deposit` function and sway the results.

- Future integration with LSTs with less than 1e18 decimals: Our team uncovered a vulnerability based on the developer's assumption that all LST tokens used as supported assets for the system will use 1e18 decimals. The developers communicated in a private discord thread that they are planning to add more supported LST assets in the future if EigenLayer expands. 
```
Gus — Yesterday at 6:38 PM
When Eigenlayer adds/accepts more LSTs, we will want to support those as well
```
If in the future additional assets are supported with a decimal of more/less than 1e18, the calculaction used in the for-loop at `LRTOracle::getRSETHPrice#L66` will return heavily skewed results and thus minting a way bigger/smaller amount than it should.

### 5.2 Centralization Risks
Parts of the functionality of the system is locked behind priviliged-role access control modifiers, functions such as adding new supported assets, setting their deposit limits, increasing the node delegator count, pause/unpause of the protocol, transfer of funds between contracts and EigenLayer strategies and oracle integration.

All of these are expected, except maybe letting users control how many of their funds, and when, are allocated to EigenLayer strategies. The codebase is still in a very early stage (especially seeing a `Withdraw` function is missing), and it might be in the plans of the developers to allow for such actions to be decided by the users, but as it stands right now, I feel like that part is rather centralized and an improvement to this would be to allow the users to choose.

## 6. Team Findings

| Severity | Findings          |
|----------|-------------------|
| Critical | 0                 |
| High     | 1                 |
| Medium   | 1                 |
| Low      | 0                 |

## Time Spent ⌛

|            |            |
|------------|------------|
| Start Date | 10.11.2023 |
| End Date   | 15.11.2023 |
| Days spent | 5 Days     |
| Hours      | 35h        |

### Time spent:
35 hours