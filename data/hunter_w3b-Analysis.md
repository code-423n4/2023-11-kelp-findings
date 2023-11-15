# Analysis - Kelp DAO | rsETH Contest

![Kelp DAO | rsETH](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2F3LECGwvBspH.0&w=256&q=75)

## Description overview of Kelp DAO Contest

![Kelp DAO](https://i.im.ge/2023/11/15/AOVmfz.kepl-Dao.png)

Kelp DAO aims to unlock liquidity, DeFi opportunities, and higher rewards for restaked assets. As a DAO, Kelp operates in a decentralized manner driven by its community. The core focus of Kelp is the development of rsETH (Liquid Restaked ETH), its flagship project. RsETH provides liquid staking for ETH deposited in the EigenLayer proof-of-stake platform. It acts as a liquid token that represents the staked ETH. This innovative approach allows holders to retain ongoing staking yields while gaining liquidity and usability of their assets in decentralized finance applications.

**The key contracts of Kelp DAO | rsETH protocol for this Audit are**:

- **LRTDepositPool.sol**: It is the main contract that users interact with to deposit assets into the Kelp DAO protocol, allowing users to easily deposit assets and receive rsETH tokens, while properly distributing those assets across contracts, the LRTDepositPool contract forms the core user interface and asset management layer for the protocol.

- **LRTConfig.sol**: The LRTConfig contract centralizes all the important configuration details of the Kelp DAO protocol.

  It stores information like:

  - Supported assets and their deposit limits
  - Contract addresses for key components
  - Token and strategy mappings

- **NodeDelegator.sol**: This contract handles the actual deposit of assets into the strategies on EigenLayer.By acting as an intermediary that directly interacts with the EigenLayer strategies, the NodeDelegator contract enables assets to flow from the deposit pool into yield/liquidity generating strategies. This is a core part of realizing the liquid staking functionality that Kelp DAO aims to provide through integration with EigenLayer.

- **LRTOracle.sol**: LRTOracle is responsible for fetching and calculating crucial exchange rates between different assets in the protocol.

  Specifically:

  - It provides the asset/ETH price by querying configured price feed contracts for each supported asset.
  - It calculates the RSETH/ETH price based on total underlying asset value deposited across the protocol and current RSETH supply.

- **RSETH.sol**: This contract represents the receipt token that users receive in exchange for assets they deposit into the protocol. By acting as the receipt token for the protocol, RSETH enables users to easily participate, while still retaining benefits of the underlying staked assets like yield/liquidity.

## System Overview

### Scope

- **LRTConfig.sol**: This contract serves as a configuration manager. It implements role-based access control, manages supported assets, their deposit limits, and associated strategies. The contract allows the addition of new supported assets, updates to deposit limits and strategies, and provides getter and setter functions for accessing and modifying various protocol parameters.

- **LRTDepositPool.sol**: LRTDepositPool contract manages the depositing of LST assets. The contract tracks the distribution of assets among the deposit pool, node delegator contracts (NDCs), and an eigen layer. Users can stake LST assets, and the contract calculates the corresponding amount of rsETH to mint based on an oracle's asset prices. It also allows the addition of node delegator contracts, asset transfers to node delegators, and updates to the maximum node delegator count.

- **NodeDelegator.sol**: The NodeDelegator facilitates the depositing of assets into strategies. It interacts with the LRT configuration and implements pausing and reentrancy protection. The contract allows the LRT manager to approve the maximum amount of an asset to the eigen strategy manager, deposit assets into strategies, and transfer assets back to the LRT deposit pool. Additionally, it provides functions to fetch asset balances and individual asset balances deposited into strategies through the node delegator.

- **LRTOracle.sol**: This contract serving as an oracle . It calculates exchange rates for assets against ETH and RSETH. The contract utilizes external price oracles through the IPriceFetcher interface and fetches asset prices, allowing users to query exchange rates.

- **RSETH.sol**: The RSETH contract is an ERC-20 token, featuring pausing functionality and role-based access controls for minting and burning. Can be associated with a specific LRT configuration contract. The contract allows for the management of roles, pausing operations, and updates to the LRT configuration.

- **ChainlinkPriceOracle.sol**: The contract is for fetching asset-to-ETH exchange rates from Chainlink price feeds, allowing dynamic updates of price oracles.

- **LRTConfigRoleChecker.sol**: The LRTConfigRoleChecker is an abstract contract facilitating role-based access control for configuration.

### Privileged Roles

Some privileged roles exercise powers over the Controller contract:

- **LRT-Manager**

```solidity
    modifier onlyLRTManager() {
        if (!IAccessControl(address(lrtConfig)).hasRole(LRTConstants.MANAGER, msg.sender)) {
            revert ILRTConfig.CallerNotLRTConfigManager();
        }
        _;
    }
```

- **LRT-Admin**

```solidity
      modifier onlyLRTAdmin() {
        bytes32 DEFAULT_ADMIN_ROLE = 0x00;
        if (!IAccessControl(address(lrtConfig)).hasRole(DEFAULT_ADMIN_ROLE, msg.sender)) {
            revert ILRTConfig.CallerNotLRTConfigAdmin();
        }
        _;
    }
```

## Approach Taken-in Evaluating The Kelp DAO

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the Kelp DAO Protocol.

    **Main Contracts I Looked At**

                LRTDepositPool.sol
                LRTConfig.sol
                NodeDelegator.sol
                LRTOracle.sol
                RSETH.sol
                ChainlinkPriceOracle.sol

    I started my analysis by examining the `LRTDepositPool.sol` contract it's important because the `LRTDepositPool` contract manages LST asset deposits, including functionalities such as tracking deposit distribution across the protocol, managing deposit limits, minting rseth tokens based on asset amounts and exchange rates, and handling node delegator contracts with features like adding to a queue, transferring assets, and updating the maximum node delegator count.

    Then, I turned our attention to the `LRTConfig.sol` contract second because it manages critical configuration parameters for the LRT and serves as a centralized repository for critical configuration parameters in the Kelp DAO protocol, managing details such as supported assets with associated deposit limits, contract addresses for key components, and mappings between tokens and their respective strategies.

    Then `NodeDelegator.sol` contract next as its pivotal role in managing the depositing and withdrawal of assets into and from strategies within the LRT protocol, So ensuring the security of operations that impact the overall protocol's integrity and user assets.

    Then audit `LRTOracle.sol`, as it provides essential functionality for determining exchange rates, enabling precise asset valuation within the protocol.

    And `RSETH` contract as it represents the ERC20 implementation for the rsETH token, requiring careful examination to ensure secure minting, burning, and proper access control roles, especially pertaining to the interaction with the LRT configuration.

2.  **Documentation Review**:

    The protocol currently lacks documentation,So went to Review [This Blogs](https://blog.kelpdao.xyz/) for a more detailed and technical explanation of Kelp DAO.

3.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

4.  **In summary, I have identified some vulnerabilities. I hope addressing these issues will enhance the security of the Kelp DAO Protocol.**

    - The deposit limit for assets can be reduced by the manager, which could lock user assets if the limit is set below existing deposits.

    - Asset strategies can be changed by any admin, but there is no logic to withdraw existing funds from the old strategy, which could result in lost funds.

    - Strategies can be set per asset, but the contract code may expect a single global strategy, potentially causing funds to be sent to incorrect strategies.

    - The total balance calculation iterates through node delegate contracts, but does not prevent reentrancy or take a snapshot, so a deposit made during iteration would be missed, giving an incorrect low total balance.

    - The asset strategy concept is defined but strategies are not actually used or passed asset data, undermining extensibility and potentially causing unexpected behaviors.

    - Token transfers using transferFrom do not check allowance first, which could cause reverts if allowance is too low.

    - Strategy function calls do not validate success or return values, so failures or invalid returns would be ignored, potentially leaving funds in invalid states.

- **Addressing these issues through strengthened input validation, error handling, and refactoring dependent relationships would help secure the contracts against exploitation or unintentional bugs.**

## Architecture Description and Diagram

Architecture of the key contracts that are part of the Kelp DAO | rsETH protocol:

This diagram illustrates the interaction among the key components of the Kelp Dao protocol.

![Kelp-Dao](https://i.im.ge/2023/11/15/A1LqyC.Kelp-Dao-Diagram.png)

- User deposits assets into the LRTDepositPool contract. An arrow from the user to the deposit pool showing the deposit.

- The LRTDepositPool contract interacts with the LRTOracle contract to get the current price of the deposited assets, represented by an arrow between them labeled "Price".

- Based on the asset prices, the LRTDepositPool then interacts with the RSETH contract to mint the equivalent amount of RSETH tokens for the user, shown by an arrow between them labeled "Mint RSETH".

- The RSETH tokens are then sent back to the user, represented by an arrow from RSETH back to the user labeled "RSETH".

- The LRTDepositPool contract then interacts with the NodeDelegator contract, shown with an arrow between them labeled "Transfer Assets".

- The NodeDelegator contract delegates the received assets to strategies on Eigenlayer, represented by an arrow pointing down to a box drawn as "Eigenlayer".

- The NodeDelegator is also able to transfer assets back to the LRTDepositPool if needed, shown with an arrow between them labeled "Transfer Back".

So in summary, it shows the core user deposit and withdrawal process, integration with oracle pricing, and delegation of assets to Eigenlayer strategies.

### Architecture Feedback

1. Compose a comprehensive and detailed documentation for the Kelp DAO protocol, covering all pertinent aspects, including its purpose, features, smart contracts, security measures, community engagement strategies, user guidelines, and development roadmap.

2. **Node Delegator**: Manages the movement of liquid staked assets to contracts for each operator, automating reward redemption for restakers.

3. Leverage zEIP-23 for formally verified external calls in NodeDelegator to Eigen for highest security assurance.

4. I strongly recommend getting formal verification done on contracts to provide the highest level of security.

5. Support multiple LST integrations from the start to benefit from diversification and maximize API compatibility for future integrations.

6. Implement EIP-1271 signature verification in deposit/withdrawal functions to authenticate calls from approved wallets, preventing front-running attacks.

## Codebase Quality

Overall, I consider the quality of the Kelp DAO codebase to be excellent. The code appears to be very mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The codebase demonstrates moderate maintainability with well-structured functions and comments, promoting readability. It exhibits reliability through defensive programming practices, parameter validation, and handling external calls safely. The use of internal functions for related operations enhances code modularity, reducing duplication. Libraries improve reliability by minimizing arithmetic errors. Adherence to standard conventions and practices contributes to overall code quality. However, full reliability depends on external contract implementations like openzeppelin. |
| **Code Comments**                        | The contracts have comments that are used to explain the purpose and functionality of different parts of the contracts but add more detailed comments for functions, state variables, and mappings for better clarity, making it easier for developers and auditors to understand and maintain the code. The comments provide descriptions of methods, variables, and the overall structure of the contract.                                                                                                                                                                                         |
| **Documentation**                        | Compose a comprehensive and detailed documentation for the Kelp DAO protocol, covering all pertinent aspects, including its purpose, features, smart contracts, security measures, community engagement strategies, user guidelines, and development roadmap.                                                                                                                                                                                                                                                                                                                                        |
| **Testing**                              | The audit scope of the contracts to be audited is 98% and it should be aimed to be 100%.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| **Code Structure and Formatting**        | The codebase contracts are well-structured and formatted. It inherits from multiple components, adhering for-example LRTDepositPool appears to be generally good. It follows best practices, uses OpenZeppelin's upgradeable contracts, incorporates access control with role-based permissions, implements pausing mechanisms, and provides clear and well-documented functions.                                                                                                                                                                                                                    |

## Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the Kelp Dao protocol. These risks encompass concentration risk in LRTConfig, LRTDepositPool, NodeDelegator risk and more, third-party dependency risk, and centralization risks arising from the existence of an “owner” role in specific contracts.

Here's an analysis of potential systemic and centralization risks in the provided contracts:

### Systemic Risks:

1. **No having, fuzzing and invariant tests could open the door to future vulnerabilities**.

2. The dynamic array nodeDelegatorQueue in LRTDepositPool contract has the potential for high gas costs if the length becomes excessively large. This could lead to transaction failures or expensive operations, especially during functions like addNodeDelegatorContractToQueue.

3. Nothing prevents node operators in LRTDepositPool contract from colluding to manipulate the system through their control over funds.

4. The maxApproveToEigenStrategyManager function in NodeDelegator contract approves the maximum amount of an asset to the Eigen Strategy Manager without a specific limit. This can pose a risk if the approval is misused or if there are vulnerabilities in the Eigen Strategy Manager contract.

5. In LRTOracle contract the getRSETHPrice function calculates the RSETH price based on the total ETH in the pool and the total supply of RSETH. If there are inaccuracies in the underlying data or if the total ETH calculation is incorrect, it can lead to incorrect RSETH prices.

6. The Protocol uses a single source (Chainlink) for fetching asset prices. If this single source is compromised, experiences downtime, or provides inaccurate data, it can impact the reliability of the entire oracle system.

### Centralization Risks:

1. Only Administrators have the authority to perform these functions.

   - Update the LRTConfig contract address (single point of control over config)
   - Add new node delegator contract addresses to the queue (centralized control over node operators)
   - Transfer assets from the deposit pool to node delegators (centralized redistribution of funds)
   - Update the maximum number of node delegators (centralized control over network size)
   - Pause the contract in emergency situations (centralized pause control)
   - Unpause the contract after being paused (centralized unpause control)
   - Update supported asset configurations like price oracles (centralized config control)
   - Update deposit limits for assets (centralized limits control)
   - Change asset strategies (centralized strategy control)
   - Upgrade the contract to new versions (centralized upgrade control)
   - Grant/revoke admin roles for future admins (centralized governance control)
   - Set/change critical addresses stored in LRTConfig (centralized dependency control)
   - Adjust any other parameters defined in LRTConfig (broad centralized config control)
   - Admin can grant/revoke privileged roles like manager at will without community input
   - All critical functions, in NodeDelegator including asset approval, depositing into strategies, and transferring assets, can only be performed by the LRT manager
   - All critical functions, in LRTOracle contract including updating price oracles, pausing/unpausing the contract, and managing the configuration, can only be performed by the LRT manager.
   - The dynamic array nodeDelegatorQueue in LRTDepositPool contract can be manipulated by the LRTAdmin role, introducing centralization risks in managing the array of node delegator contracts.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the Kelp Dao protocol.**

## Conclusion

In general, the Kelp Dao project exhibits an interesting and well-developed architecture we believe the team has done a good job
regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from
potential malicious use cases. Additionally, it is recommended to add and improve the documentation and comments in the code to enhance understanding and collaboration among developers. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.


### Time spent:
20 hours