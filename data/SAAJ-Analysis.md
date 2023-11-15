
# Introduction
This analysis reports on the ```Kelp Dao``` contest which is developed as product to stake assets. The assets when staked will be converted to a token named ```rsETH```. The protocol uses oracle that calculates the exchange rate of assets. It fetches prices from different price oracles for each asset and calculates the total value of assets in the pool.

Every asset is first approve and deposit into strategies, transfer assets back to the LRT deposit pool, and fetch the balance of all assets staked in the eigen layer through this contract.

The staked mechanism is based on fetching the total amount of an asset, and gets the asset amount distribution data among the deposit pool, NDCs, and the Eigen layer.



# Mechanism Review
This section looks at working mechanism of contracts in scope and discussed the workflow for each contract.

**ChainlinkPriceOracle** contract interacts with Chainlink price feeds to fetch the exchange rate of assets. The contract has a state variable ```assetPriceFeed``` which is mapping from an asset address to a price feed address. ```getAssetPrice``` function fetches the exchange rate of a given asset by calling the latestAnswer function on the Chainlink price feed associated with the asset. ```updatePriceFeedFor``` function checks if the new price feed address is non-zero and then updates the assetPriceFeed mapping.

**LRTConfig** contract manages various aspects of the application, such as supported assets, deposit limits, and strategies for each asset. Here's a breakdown of its main functionalities. Assets can be added to this list using the ```addNewSupportedAsset``` function and the deposit limit for each asset can be updated using the ```updateAssetDepositLimit``` function.

**LRTConfigRoleChecker** as an abstract contract handles role checks for a LRT (Liquidity Reward Token) configuration. ```updateLRTConfig``` function updates the lrtConfig state variable and also checks that the provided address is not zero before updating the lrtConfig and emitting the UpdatedLRTConfig event.

**LRTDepositPool** contract handle depositing of assets into the protocol and the management of node delegators. The contract checks if the asset is supported and if the deposit amount is within the current limit. If the checks pass, the contract transfers the asset from the user to itself and mints a corresponding amount of ```rsETH``` tokens for the user ```_mintRsETH``` function. The amount of rsETH minted is calculated based on the asset amount and the asset's exchange rate from an oracle. It provides distribution of an asset among the deposit pool, node delegator contracts (NDCs), and the eigen layer through ```getAssetDistributionData```. The ```getAssetDistributionData``` function returns the amount of the asset in the deposit pool, the total amount in all NDCs, and the amount staked in the eigen layer through all NDCs.

**LRTOracle** contract is used for asset price Fetching by the ```getAssetPrice``` function fetches the price of a given asset by calling the ```getAssetPrice``` function of the asset's price oracle. ```getRSETHPrice``` function calculates the price of RSETH based on the total supply of RSETH and the total amount of ETH in the pool. ```updatePriceOracleFor``` function allows the LRTManager to update the price oracle for a given asset.

**NodeDelegator** is responsible for managing the depositing of assets into strategies. ```maxApproveToEigenStrategyManager``` function approves the maximum amount of a specific asset to the Eigen Strategy Manager. ```depositAssetIntoStrategy``` function deposits a specific asset into its strategy. The function first retrieves the strategy for the asset from the LRT config, then retrieves the balance of the asset in the contract, and finally calls the ```depositIntoStrategy``` function of the Eigen Strategy Manager with strategy, token and balance. 
```transferBackToLRTDepositPool``` function transfers a specific amount of a specific asset back to the LRT deposit pool. If the transfer fails, it reverts the transaction with a TokenTransferFailed error. ```getAssetBalances``` function fetches the balance of all assets staked in the Eigen layer through this contract. It returns an array of assets and their corresponding balances.

**RSETH** contract have various functionality like minting, burning, pausing and unpausing. ```updateLRTConfig``` function allows the DEFAULT_ADMIN_ROLE to update the lrtConfig address. It checks that the new address is non-zero, updates the lrtConfig, and emits an UpdatedLRTConfig event. The contract defines roles like ```MINTER_ROLE``` and ```BURNER_ROLE``` which are used to control access to the minting and burning functions respectively.

# Codebase Quality Analysis

There are some pitfalls identified in the codebase related to quality of the code which can affect overall smooth working and efficiency of the ```Kelp Dao``` project.
- The codebase have some parts of the code that are well-commented, others are not. Adding more comments, especially for complex functions, would improve the code's readability and maintainability.
- Some functions do not validate their inputs like the initialize function in the ```RSETH``` contract does not check if the admin or ```lrtConfigAddr``` addresses are valid (non-zero) addresses.
- The ```getAssetPrice``` function uses ```latestAnswer``` from the Chainlink Aggregator contract. This could potentially return stale data if the latest round is not completed yet. It might be better to use getRoundData and handle the case when the round is not completed yet.
-  Some functions changes the state of the contract do not emit events. Events are useful for off-chain services to track the state changes of a contract.
- Some functions that change the state of the contract do not have access control modifiers. This could potentially allow anyone to call these functions and change the state of the contract.

# Centralization Risks
There exists several instance of controlled mechanism to a single or group of entity which creates clear issue of centralisation for the project which are discussed as below:
- In all the contracts, there are functions that can only be called by an admin or a manager. This includes functions to pause and unpause the contract, update configurations, and add new supported assets. 
- In ```NodeDelegator``` and ```LRTDepositPool``` contracts, there are functions that handle the depositing and transferring of assets. These functions can only be called by the LRT manager. This centralizes control over the assets in the contract.
- In ```ChainlinkPriceOracle``` contract price feed for each asset can be updated by the LRT manager. This have the potentially to allow for manipulation of asset prices.
- In ```RSETH``` contract there are roles for minting and burning tokens. If these roles are not properly decentralized, it could lead to manipulation of the token supply.
- ```LRTConfig``` contract have strategies for assets can be assigned only by the admin. This could potentially allow for manipulation of asset strategies.

# Systemic Risks
Systemic risk identified for this project is related to risk that a failure in one part of the system could lead to a failure in the entire system. The potential systemic risks identified for project of ```Kelp Dao``` are discussed as below:
- The contracts depend on external contracts such as the OpenZeppelin contracts and Chainlink Price Oracle. If there are any issues or vulnerabilities in these external contracts, it could affect the entire system.
- The contracts have roles like LRTManager and LRTAdmin that have special permissions. If the private keys of these accounts are compromised, it could lead to a systemic failure.
- The project uses the Upgradeable contracts and if the upgrade process is not secured properly, it could lead to an attacker being able to change the contract logic.
- The project rely on the Chainlink Price Oracle for asset pricing, if the oracle fails or provides incorrect data, it could affect the entire system.

# Architecture Recommendations
The recommendations are mainly made in context of improving the working mechanism of the project and ensure more convenience to the users. The recommendation will also help other developers to understand the codebase and make changes accordingly with standard safe solidity practice.
First recommendation is made for instances of utility functions are used across multiple contracts. Consider moving these utility functions into a library. This can help in reducing the overall gas cost and make the code more understandable and easier to manage.

Recommendation is also made for some contracts use revert statements, while others do not. It's recommended to have consistent error handling across all contractsas it will make it easier to debug and understand the code.

There are some comments in the code and it would be beneficial to have more detailed comments explaining the logic of complex functions. This will make the code easier to understand for new developers and help in maintaining the code in the future.

The recommendation is made to effectively use the upgradeability process . It's important to ensure that all state variables are added to the end of the contract to prevent issues with storage layout.

The project use OpenZeppelin's ```AccessControl``` for role-based access control. It's important to review the role assignment and usage of these roles across the contracts to ensure only the necessary permissions are granted and that roles can't be abused.

# Conclusion
This analysis provide details on working mechanism, short fall in code quality and the risk identified for the codebase of the ```Kelp Dao``` project. Recommendation is made to improve any short fall that might impact the security and smooth working of the project in the long term.

The time spent on this project is around 25 hours which helps in identified 03 mediums issues that must be addressed to ensure no risk to the funds or workflow of the project exists that may impact the protocol in a negative manner.


### Time spent:
025 hours