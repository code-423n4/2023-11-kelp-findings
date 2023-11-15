| List | Head | Details |
| --- | --- | --- |
| 1   | introduction to TheKelp DAO | introduction to  The  Kelp DAO |
| 2   | Audit approach | Process and steps I followed |
| 3   | Architecture recommendations | some recommendations related to the architecture |
| 4   | Contracts | explanation about Contracts in the scope |
| 5   | Test analysis | Test analysis percentage |
| 6   | Codebase Quality | how was the quality of the codebase |
| 7   | How could they have done it better? | Some best code practice suggestions |
| 8   | Possible Systemic Risks | The possible systemic risks based on my analysis |
| 9   | Centralization risks | Concerns associated with centralized systems |
| 10  | Security Approach | Security Approach of the Project |
| 11  | Gas Optimizations | Details about my gas optimizations findings and gas savings |
| 12  | Documentation | how was the Documents that they provided for us |
| 13  | Recommendations | some recommendations to the Kelp DAO \| rsETH teams |
| 14  | Conclusion | Conclusion |
| 15  | Time spent on analysis | The Over all time spend for this reports |

## Introduction

Kelp is a collective DAO designed to unlock liquidity, DeFi and higher rewards for restaked assets. This is coupled with the convenience of a single liquid restaked token for accepted LSTs. rsETH will be further developed under the Kelp brand. More details to follow soon.

&nbsp;**What is rsETH?**  
 rsETH is a liquid restaked token that empowers users to receive Ethereum staking and restaking rewards while retaining liquidity and the ability to participate in DeFi.

## Audit Approach

1.  **Initial Scope and Documentation Review**: Thoroughly went through the Contest Readme File and Blog to understand the protocol's objectives and functionalities.
    
2.  **High-level Architecture Understanding**: Performed an initial architecture review of the codebase by going through all files without going into function details.
    
3.  **Test Environment Setup and Analysis**: Set up a test environment and execute all tests. Additionally, use Static Analyzer tools like Slither to identify potential vulnerabilities.
    
4.  **Comprehensive Code Review**: Conducted a line-by-line code review focusing on understanding code functionalities.
    
    - **Understand Codebase Functionalities**: Began by going through the functionalities of the codebase to gain a clear understanding of its operations.
    - **Identify Access Control Issues**: Thoroughly examine the codebase for any access control problems that might allow unauthorized users to execute critical functions.
    - **Evaluate Function Execution Order**: Checked random sequence of function executions to ensure the protocol's logic cannot be disrupted by changing the order of calls.
    - **Assess State Variable Handling**: Identify state variables and assess the possibility of breaking assumptions to manipulate them into exploitable states, leading to unintended exploitation of the protocol's functionality.
5.  **Report Writing**: Write a Report by compiling all the insights I gained throughout the line-by-line code review.
    

## Architecture recommendations

I suggest having the contract under audit equipped with a complete set of NatSpec detailing all input and output parameters pertaining to each functions although much has been done in specifying each @dev. Additionally, a thorough set of flowcharts, and if possible a video walkthrough, could mean a lot to the developers/auditors/users in all areas of interests.

## Contracts

- LRTConfig.sol: is a configuration contract for a system   to manage various Ethereum-based assets. It uses OpenZeppelin's `AccessControlUpgradeable` for role-based access control and is designed to be upgradeable.
    
- LRTDepositPool.sol:
    
    It handles the deposit of assets, specifically LST tokens, into the protocol. The contract uses OpenZeppelin's `PausableUpgradeable` and `ReentrancyGuardUpgradeable` contracts to prevent reentrancy attacks and to allow pausing of the contract. It also uses a custom library `UtilLib` and a custom contract `LRTConfigRoleChecker` for role-based access control
    
- NodeDelegator.sol: manages the depositing of assets into various strategies. It uses OpenZeppelin's Pausable and ReentrancyGuard contracts to prevent reentrancy attacks and to pause/unpause contract functions.
    
- LRTOracle.sol: it is used to calculate the exchange rate of assets. It imports several libraries and interfaces, including OpenZeppelin's `PausableUpgradeable` contract which allows the contract to be paused and unpaused by authorized roles.
    
- RSETH.sol:
    
    This contract is based on the ERC20 standard, which is a common standard for fungible tokens on the Ethereum blockchain. The contract also includes features for pausing and access control, which are provided by the OpenZeppelin library.
    
    The contract defines two roles: a minter and a burner. The minter can create new tokens, and the burner can destroy tokens. These roles are controlled by the AccessControl feature of the contract.
    

## Test analysis

The audit scope of the contracts to be reviewed is 98%.

## Codebase Quality

- **Modular Design:** The code is organized into different contracts, each responsible for specific functionality. This makes the codebase easier to understand and maintain.
    
- **Clear Comments:** There are comments throughout the code that explain the purpose of functions, variables, and sections. This enhances readability and comprehension.
    
- **Access Control:** The contracts implement access control using modifiers like `onlyRole`. This ensures that certain functions can only be called by authorized parties.
    
- **Events:** Events are used to provide transparency and enable easier tracking of important contract interactions.
    

## How could they have done it better?

While the provided code seems to be functional, there are always areas for improvement to ensure better security, efficiency, and readability in the contract and the protocol.

- Here are some potential areas for improvement:
    
- **Comments and Documentation**: Provide thorough comments and documentation throughout the code to explain the purpose, functionality, and any potential gotchas of each component.
    
- **Input Validation**: Validate and sanitize all user inputs to prevent unexpected behavior or attacks. For example, checking that input amounts are positive, non-zero, and within reasonable bounds.
    
- **Consistent Naming Conventions**: Use consistent and descriptive variable and function names to make the code more understandable.
    
- **Gas Efficiency**: Optimize the code for gas efficiency. This involves minimizing unnecessary computations, storage, and external calls to save on transaction costs.
    

## Possible Systemic Risks:

- **Algorithmic Complexity**: The contract involves complex mathematical calculations and interactions. Errors in these calculations could lead to incorrect outcomes, affecting the functioning of the contract and potentially leading to unexpected losses for users.
    
- **Rounding Errors**: Rounding errors can accumulate over time due to frequent calculations involving floating-point arithmetic. These errors could lead to discrepancies between expected and actual token balances, causing loss of funds.
    
- **Front-Running and Manipulation**: Complex financial systems can be susceptible to front-running attacks where malicious actors place transactions ahead of others to gain an advantage. The evolving nature of the bonding curve could provide opportunities for manipulation.
    
- **Unintended Consequences**: The interactions between different parts of the contract could lead to unintended consequences, negatively impacting users.
    

## Centralization Risk

No centralization Risk

## Security Approach of the Project

- **Minimize Complexity**: Keep the smart contract logic as simple as possible to reduce the attack surface. Complex systems are more prone to bugs and vulnerabilities.
    
- **Separation of Concerns**: Divide the contract's functionalities into smaller, modular components. This can help isolate vulnerabilities and make the contract easier to audit.
    
- **Secure Coding Guidelines**: Enforce secure coding practices among developers. Follow best practices for Solidity development to minimize the risk of introducing vulnerabilities.
    

Remember that security is an ongoing process, and regular reviews, updates, and improvements are essential to stay ahead of emerging threats. Collaborating with experienced blockchain security professionals and conducting frequent security assessments can significantly enhance the security posture of the project.

## Gas Optimization

\[G‑01\] Save gas by preventing zero amount in mint() and burn()

\[G-02\] Amounts should be checked for 0 before calling a transfer

\[G-03\] Use constants instead of type(uintx).max

\[G-04\] use Mappings Instead of Arrays

\[G-05\] The result of a function call should be cached rather than re-calling the function

\[G-06\] Use hardcode address instead `address(this)`

\[G-07\] When possible, use assembly instead of `unchecked{}`

\[G -08\] Compute known value `off-chain`

&nbsp;\[G-09\] Use assembly to emit an event To efficiently emit events,

## **Documentation**

- Unfortunately, the only documentation very challenging to review in terms of explaining its functionality, parameter usage, purpose, and overall architecture. However, here are some suggestions that could further enhance the clarity and understanding of the documentation:
    
- **Visual Examples**: Include diagrams or visual aids to help readers understand concepts better.
    
- **Practical Examples**: Provide specific numerical examples to help readers apply calculations and parameters in real-world scenarios.
    
- **Detailed Use Cases**: Expand use cases to demonstrate how the contract applies in real life.
    
- **Highlighted Key Terms**: Boldly highlight technical terms when first mentioned to help readers identify important concepts.
    

## Recommendations:

To enhance the security posture of the Kelp DAO | rsETH several measures can be considered. Implementing extra security mechanisms like timelock delay consensus or requiring multisig approvals for critical changes can provide an extra layer of protection. Introducing a decentralized validation process for significant upgrades could help ensure that changes are authorized by a broader consensus. Additionally, documentation should be improved to provide comprehensive insights into roles, responsibilities, and interactions within the system. Thorough testing, particularly focused on upgrade and timelock processes, is essential for identifying and mitigating potential vulnerabilities. For further assurance, a formal audit by a reputable smart contract security firm is highly recommended.

## Conclusion

In general, the `Kelp DAO | rsETH` exhibits an interesting and well-developed architecture we believe the team has done a good job regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from potential malicious use cases. Additionally, it is recommended to improve the documentation and comments in the code to enhance understanding and collaboration among developers. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.

## Time Spent on the Audit

- 17 hours

&nbsp;

&nbsp;

&nbsp;

### Time spent:
17 hours