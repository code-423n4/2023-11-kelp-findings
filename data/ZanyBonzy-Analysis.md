### **Protocol Overview**

  Kelp is a collective DAO designed to unlock liquidity, DeFi and higher rewards for restaked assets. It aims to address the potential challenges of restaking, such as high gas fees, liquidity constraints, and node operator discovery by introducing a new Liquid Restaked Token which is issued for restaked ETH. At the heart of the protocol is the `rsETH` token which is an LRT that provides liquidity to illiquid assets deposited in restaking platforms.
  
***
### **Codebase Analysis**

  As the codebase is still in the development phase, it's not very large. However, it's well-structured, and functions are broken down into small, easy-to-understand bits - not too cyclomatically complex. The codebase (in scope) comprises a total of 17 contracts (8 interfaces and 9 actual contracts) and about 500 SLoC. The codebase mostly uses inheritance, defines no structs, and includes 2 external calls (to Chainlink oracle and to EigenLayer). The overall test line coverage is about 98%. The core contracts are designed to be upgradeable; therefore, the protocol uses OZ upgradeable imports.
  
- The contracts in scope (other than the interfaces) are 
  1. `LRTConfig.sol` is the protocol's configuration contract	that also stores the protocols' contract addresses.
  2. `LRTDepositPool.sol` is the contract through which users deposit funds to the protocol, and have their funds transferred to the `NodeDelegator` contracts.
  3. `NodeDelegator.sol` receives funds from the depositpool and delegates them to the eigenlayer strategy.
  4. `LRTOracle.sol` fetches prices of LST tokens from oracles.
  5. `RSETH.sol` is the	receipt token a user receives upon depositing in `LRTDepositPool.sol`.
  6. `ChainlinkPriceOracle.sol` is integrates chainlink oracles in LRTOracle.
  7. `LRTConfigRoleChecker.sol` performs role checks.
  8. `UtilLib.sol is the helper` function library. It handles 0x0 address checks.
  9. `LRTConstants` is contains the global constant variables.

***
## **Architecture Review**

  Restakers stake their `ETH`/liquid tokens by transferring them into the `depositPool` contract. This mints them the `rsETH` tokens in return, based on the current exchange rate. The staked `ETH`/liquid tokens are then transferred to the `NodeDelegator` contract. From here, they are delegated to the Eigenlayer protocol through predefined strategies. It is important to note that as of now, there is no way to withdraw directly from the protocol, but the team has confirmed that it is in the works.
  
***
## **Protocol Risks**
- Some of the risks that can affect the protocol include
  1. Risk from centralization, malicious or compromised managers and admins.
  2. Any issues with the EigenLayer can have a ripple effect on the protocol.
  3. Compromised smart contracts in the protocol.
  4. Issues with ChainLink oracle can affect asset prices and consequently, amount of `rsETH` tokens minted.
  5. Non-standard ERC20 token types, if they eventually get integrated.
  6. Risks from third-party contract dependencies.

***
 
 ## **Approach to Audit**
  - We approached the audit in four steps.
    1. We reviewed the provided documents, blogs, and also noted other explanations provided by the developers on Discord.
    2. We ran the contracts through Slither and compared the generated report to the bot reports. From this, we weeded out the known issues and false positives.
    3. We carefully reviewed the codebase, ran provided tests, and worked out various attack vectors. We also tested the functions' logic to make sure they work as intended.
    4. We noted our findings and created the analysis report.
 ***    

## **Recommendations**
-  
  1. Consider implementing a backup oracle to serve as an alternative to mitigate the risk of Chainlink failure.
  2. Due to the limited range of ERC20 tokens supported by EigenLayer, the protocol gets a pass for not performing safe ERC20 operations. But depending on the token types to be integrated, it's safer to use them. Also, approving `type(uint256).max` might not work for some tokens (see NodeDelegator [L45](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L45)). Care should be taken when adding tokens with multiple addresses, etc.
  3. Consider creating a proper documentation through which users and developers can better understand the contracts and functions.
  4. Consider introducing a `multiplier` constant to deal with possible precision loss when performing division operations.
  5. Consider reformatting the functions to better align with the Solidity style guide and introducing NatSpec comments.

  ***
## **Conclusion**
  In general, the protocol seems interesting and we're anticipating what the team has in store next. The architecture is solid, the codebase is well written, and most of the easy-to-overlook safety measures are actually implemented. However, the identified risks need to be mitigated, and provided recommendations should be taken into consideration. Constant upgrades and audits should be invested in to ensure the security of the protocol.




### Time spent:
20 hours