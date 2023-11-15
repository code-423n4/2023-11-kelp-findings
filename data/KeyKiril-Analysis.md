# Overview

Kelp DAO is building a Liquid Restaked Token (LRT) designed for seamless liquidity and flexibility in DeFi. An LRT is a liquid token that provides liquidity to illiquid assets deposited into restaking platforms, such as EigenLayer. `rsETH` is a liquid restaked token that empowers users to receive Ethereum staking and restaking rewards while retaining liquidity and the ability to participate in DeFi.

Kelp DAO achieves its goal mainly through five smart contracts in scope:

# Scope

**src/LRTConfig.sol** - An upgradeable contract which is responsible for storing the configuration of the Kelp product. It is also responsible for storing the addresses of the other contracts in the Kelp product.

**src/LRTDepositPool.sol** - An upgradeable contract which receives the funds deposited by users into the Kelp product. From here, the funds are transferred to NodeDelegators contracts.

**src/NodeDelegator.sol** - These are contracts that receive funds from the LRDepositPool and delegate them to the EigenLayer strategy. The funds are then used to provide liquidity on the EigenLayer protocol. It is also an upgradeable contract.

**src/LRTOracle.sol** - An upgradeable contract which is responsible for fetching the price of Liquid Stasking Tokens tokens from oracle services.

**src/RSETH.sol** - Receipt token for depositing into the Kelp product. It is an upgradeable ERC20 token contract.


# Approach Taken in evaluating the codebase

In analyzing the codebase I've took the following steps:

**1. Architecture Review**  
    Reading available documentation to understand the architecture and main functionality of Kelp DAO and EigenLayer.

**2. Compiling code and running provided tests**

**3. Detailed Code Review**  
    A thorough code review of the contracts in scope as well as relevant parts of third party contracts (mainly Eigenlayer contracts).

**4. Security Analysis**  
    Defining the main attack surfaces of the codebase i.e. Safe funds exfiltration, unauthorized transaction executions etc.


# Mechanism Review

Kelp DAO's architecture includes several mechanisms working in unison:

## 1. "LRTConfig"
1.	Inheritance and Imports:
•	The contract inherits from the AccessControlUpgradeable contract provided by the OpenZeppelin library, which enables access control features.
•	It imports various utility libraries and interfaces, such as UtilLib, LRTConstants, and ILRTConfig.
2.	State Variables:
•	The contract defines several state variables, including mappings for token and contract addresses, lists of supported assets, deposit limits per asset, and asset strategies.
3.	Constructor and Initialization:
•	The contract has an initialize function that serves as the constructor. It sets initial values for key addresses and roles. It also grants the DEFAULT_ADMIN_ROLE to the specified admin address.
4.	Modifiers:
•	The onlySupportedAsset modifier ensures that certain functions can only be executed for supported assets.
5.	Functions:
•	The contract includes functions for adding new supported assets, updating deposit limits, updating asset strategies, and setting the rsETH contract address.
•	There are private functions for internal use, such as adding a new supported asset and setting token or contract addresses.
•	Getter functions allow external contracts to retrieve information, including the LST token address, contract address, and the list of supported assets.
6.	Events:
•	The contract emits events, such as "AddedNewSupportedAsset," "AssetDepositLimitUpdate," "SetToken," and "SetContract," to log important changes on the blockchain.
7.	Access Control:
•	The contract uses the OpenZeppelin access control mechanism, granting roles like MANAGER and DEFAULT_ADMIN_ROLE to specific addresses. Access to certain functions is restricted based on these roles.


## 2. "LRT Deposit Pool"
1.	Contract Inheritance:
•	The contract inherits from PausableUpgradeable and ReentrancyGuardUpgradeable, which provide functionality for pausing and guarding against reentrancy attacks.
2.	Initialization:
•	The contract uses the initializer pattern to initialize its state variables, making use of the OpenZeppelin __Pausable_init and __ReentrancyGuard_init functions.
3.	State Variables:
•	maxNodeDelegatorCount: Represents the maximum count of node delegators.
•	nodeDelegatorQueue: An array storing addresses of node delegator contracts.
4.	Functions:
•	View Functions:
•	getTotalAssetDeposits: Calculates the total assets deposited in the protocol.
•	getAssetCurrentLimit: Retrieves the current deposit limit for an asset.
•	getNodeDelegatorQueue: Returns an array of node delegator contract addresses.
•	getAssetDistributionData: Provides asset amount distribution data among deposit pool, NDCs, and EigenLayer.
•	getRsETHAmountToMint: Calculates the amount of rsETH to mint for a given asset amount.
•	Write Functions:
•	initialize: Initializes the contract with a reference to the LRT config.
•	depositAsset: Allows users to stake LST (or other supported assets) into the protocol.
•	_mintRsETH: Private function to mint rsETH based on asset amount and exchange rates.
•	addNodeDelegatorContractToQueue: Allows adding new node delegator contract addresses to the queue.
•	transferAssetToNodeDelegator: Transfers assets from the deposit pool to a specified node delegator.
•	updateMaxNodeDelegatorCount: Allows updating the maximum count of node delegators.
•	pause and unpause: Functions to pause and unpause the contract.

## 3. "Node Delegator"
1.	Import Statements: The contract imports various libraries and interfaces from external sources, including OpenZeppelin's PausableUpgradeable and ReentrancyGuardUpgradeable.
2.	Contract Initialization: The contract uses the OpenZeppelin initializer pattern and has an initialize function that sets the LRT config address and initializes the Pausable and ReentrancyGuard features.
3.	Modifiers Usage:
•	The contract uses the onlySupportedAsset modifier to ensure that only supported assets can be deposited.
•	The onlyLRTManager and onlyLRTAdmin modifiers are used to restrict certain functions to specific roles.
4.	Functions:
•	maxApproveToEigenStrategyManager: Allows the LRT manager to approve the maximum amount of an asset to the Eigen Strategy Manager.
•	depositAssetIntoStrategy: Allows the LRT manager to deposit an asset into its corresponding strategy.
•	transferBackToLRTDepositPool: Allows the LRT manager to transfer assets back to the LRT deposit pool.
•	getAssetBalances: Returns the balances of assets deposited into strategies by the NodeDelegator.
•	getAssetBalance: Returns the balance of a specific asset deposited into a strategy.
•	pause and unpause: Allows the LRT manager to pause and unpause the contract.
5.	State Variables:
•	The contract utilizes state variables to store the LRT config address.
6.	Initialization Safety: The contract uses the __Pausable_init and __ReentrancyGuard_init functions for proper initialization.

## 4. "LRT Oracle"
1.	Imports: The contract imports several external libraries, interfaces, and contracts such as UtilLib, LRTConstants, LRTConfigRoleChecker, IRSETH, IPriceFetcher, ILRTOracle, ILRTDepositPool, INodeDelegator, and OpenZeppelin's PausableUpgradeable.
2.	State Variables:
•	assetPriceOracle: A mapping that associates asset addresses with their respective price oracle addresses.
3.	Constructor and Initialization:
•	The constructor is marked as unsafe for upgrades, and it disables the initializers.
•	The initialize function is used for initializing the contract, specifically setting the LRT configuration address.
4.	View Functions:
•	getAssetPrice: Retrieves the exchange rate of a given asset against ETH using a priceFetcher interface.
•	getRSETHPrice: Calculates the exchange rate of RSETH against ETH based on the total supply of RSETH and the total value of supported assets in the deposit pool.
5.	Write Functions:
•	updatePriceOracleFor: Allows the LRTManager to add or update the price oracle for a supported asset.
•	pause and unpause: Allows the LRTManager to pause and unpause the contract, respectively.

## 5. "RSETH"
1.	Inheritance:
•	The contract inherits from several OpenZeppelin contracts, including ERC20Upgradeable, PausableUpgradeable, and AccessControlUpgradeable. This leverages well-tested and widely used code for ERC-20 functionality, pausability, and role-based access control.
2.	Roles:
•	The contract defines two roles using bytes32 constants: MINTER_ROLE and BURNER_ROLE. These roles are intended for minting and burning operations, respectively.
3.	Initialization:
•	The contract is upgradeable and uses the OpenZeppelin Initializable contract for initialization.
•	The initialize function is used to set up the contract, including setting the token name and symbol, initializing the access control, pausability, and configuring the initial admin and LRT (presumably an external configuration) addresses.
4.	Minting and Burning:
•	The mint and burnFrom functions are responsible for minting and burning rsETH tokens. They are restricted to addresses with the MINTER_ROLE and BURNER_ROLE, respectively, and are also subject to the contract not being paused.
5.	Pausability:
•	The contract implements pausability using the OpenZeppelin PausableUpgradeable contract. Pausing and unpausing can be done by the LRT config manager and the rsETH admin, respectively.
6.	LRT Config:
•	The contract interacts with an external contract of the ILRTConfig interface, presumably for configuration purposes. The LRT config address can be updated by the rsETH admin.
7.	Events:
•	The contract emits events like UpdatedLRTConfig to log important state changes.
8.	Modifiers:
•	The contract uses the onlyRole and whenNotPaused modifiers to control access to certain functions based on roles and the contract's pause state.
9.	Safety Checks:
•	Various safety checks using the UtilLib library are included to ensure that critical addresses are not zero addresses during initialization and LRT config updates.

# Recommendations

Based on the above mentioned mechanism review I have concluded the following list of recommendations:

### "LRTConfig.sol"
•	Verify that the access control mechanism and roles are appropriately configured.
•	Review the safety checks to confirm that they adequately protect against potential vulnerabilities.
•	Consider incorporating additional documentation for improved code readability.

### "LRTDepositPool.sol"
1.	Token Transfer Safeguards:
•	Implement additional checks and safeguards for external token transfers, especially in functions like depositAsset and transferAssetToNodeDelegator.
2.	Iterative Node Delegator Queue Operations:
•	Consider making node delegator queue operations non-reentrant and carefully review unchecked iteration.
3.	Gas Limit Considerations:
•	Be mindful of gas limits, especially when adding multiple node delegator contracts to the queue in addNodeDelegatorContractToQueue.

### "NodeDelegator.sol"
1.	Gas Optimization: The use of unchecked in the getAssetBalances function suggests a gas optimization technique. However, its usage should be carefully considered to avoid unintended consequences.
2.	Overall Impression: The contract seems to be focused on managing the deposit of assets into strategies. It relies on the LRT configuration for various contract addresses and uses OpenZeppelin contracts for certain security features.

### "LRTOracle.sol"
1.	Safe Math:
•	The code does not explicitly use SafeMath for arithmetic operations, which may lead to vulnerabilities if not handled carefully. Consider using `SafeMath` or similar libraries for arithmetic operations.
UtilLib, LRTConstants, and LRTConfigRoleChecker to ensure that the entire system operates as intended. Additionally, consider incorporating the latest Solidity version for improved security features and optimizations.

### "RSETH.sol"
1.	Upgrade Safety:
•	The contract includes an unsafe-allow constructor comment, which suggests that it might be part of an upgradeable contract system. Developers should be cautious with upgradeable contracts due to potential security considerations.
In general, the contract follows best practices, utilizing well-established libraries from OpenZeppelin and implementing access control, pausability, and upgradeability. Additionally, the functionality and security of the ILRTConfig interface and its implementations need to be verified separately.

### Time spent:
20 hours

### Time spent:
20 hours