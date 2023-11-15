# Analysis- Kelp DAO

## Overiew
A collective DAO designed to unlock liquidity, DeFi and higher rewards for restaked assets through liquid restaking.

#### LRTConfig
This contract is used to manage the configuration of the LRT (Lido Rocket Pool) protocol. It stores the addresses of the core contracts, the list of supported assets, and the deposit limits for each asset. It also provides functions for adding new supported assets, updating deposit limits, and updating asset strategies.

#### LRTDepositPool
This contract is used to manage the deposit pool for Lido Rocket Pool (LRT) tokens. It allows users to deposit LST tokens and mint rETH tokens in exchange. It also allows the LRT admin to add new node delegator contracts and transfer assets to them. The contract can be paused or unpaused by the LRT admin.

#### NodeDelegator
The NodeDelegator contract is responsible for depositing assets into strategies. It can also transfer assets back to the LRT deposit pool.

#### LRTOracle
The LRTOracle contract is responsible for calculating the exchange rate of assets. It does this by reading from the priceFetcher interface, which may fetch the price from any supported oracle.

#### RSETH
The rsETH token contract is an ERC20 token contract that is used to represent the rsETH token. The rsETH token is a staked ETH token that is used to earn rewards on staked ETH.

## Architecture Recommentations

### Implementing Supported Asset Removal in the LRT Token Ecosystem

It is currently not possible to remove supported assets once they have been added to the ``supportedAssetList``

Implementing an asset removal mechanism in your smart contract can be a prudent decision, especially if there's a possibility that certain assets might need to be delisted or removed for various reasons.

```solidity

function _addNewSupportedAsset(address asset, uint256 depositLimit) private {
        UtilLib.checkNonZeroAddress(asset);
        if (isSupportedAsset[asset]) {
            revert AssetAlreadySupported();
        }
        isSupportedAsset[asset] = true;
        supportedAssetList.push(asset);
        depositLimitByAsset[asset] = depositLimit;
        emit AddedNewSupportedAsset(asset, depositLimit);
    }

```

#### Why need asset removal Mechanism ?

- Financial and crypto markets are dynamic, with assets frequently changing in terms of compliance, popularity, and viability. A removal mechanism allows your contract to adapt to these changes efficiently
- Certain assets might become risky due to volatility, security issues, or other concerns. Removing such assets can be crucial for protecting the contract and its users from potential losses or harm
-  If an asset is found to have technical issues or vulnerabilities, removing it can prevent these problems from affecting the broader system
-  In cases where an asset is used for abusive or manipulative practices, being able to remove it can safeguard the ecosystem's health and the interests of its users

#### Mitigation
Create a function to remove an asset.


```solidity

/// @dev private function to remove a supported asset
/// @param asset Asset address
function _removeSupportedAsset(address asset) private {
    require(isSupportedAsset[asset], "Asset not supported");

    isSupportedAsset[asset] = false;
    _removeAssetFromList(asset);
    delete depositLimitByAsset[asset];

    emit RemovedSupportedAsset(asset);
}

```

### Consider implementing  oracle redundancy

oracle redundancy in the context of blockchain and smart contracts refers to the utilization of multiple oracles to provide price feeds or other data sources. By employing multiple oracles, the protocol can achieve greater reliability and accuracy in the data it receives. This is crucial for smart contracts that rely on accurate data to execute logic and make decisions.

Oracle redundancy mitigates this risk by using multiple oracles to provide the same data. The protocol can then aggregate the data from multiple sources and take a consensus approach to determine the final value. This ensures that the data is more likely to be accurate and reliable, even if one or more oracles fail or provide incorrect information.

consensus mechanisms that can be used for oracle redundancy

``Median``: The median is the middle value in a sorted list of data points. By taking the median of the data from multiple oracles, the protocol can reduce the impact of any outliers or inaccurate values.

``Mean``: The mean is the average of all data points. While the mean is a common measure of central tendency, it can be sensitive to outliers. Using the mean for oracle redundancy may require additional weighting or filtering mechanisms to account for outliers.

``Weighted Average``: A weighted average is an average in which each data point is given a weight. The weights can be based on the oracle's reputation, historical accuracy, or other factors. Using weighted averages can help give more credence to more reliable oracles.

``Multi-oracle threshold`` : This approach requires a certain number of oracles to agree on a value before it is considered valid. This can help prevent the system from accepting a value that is based on too few or unreliable sources.

``Reputation system`` : A reputation system can be used to track the performance of each oracle and give more weight to oracles that have a proven track record of accuracy.

## Systemic Risks

``Scalability limitations`` : System's architecture is not designed for scalability, it may struggle to handle a large number of users or transactions. This could lead to congestion, delays, or even system outages.

``Economic security risks`` : The system's value is tied to the underlying assets and market conditions. Fluctuations in asset prices or market downturns could lead to significant losses for users.

``Operational risks`` : The system relies on a network of interconnected components and external services. Failures or disruptions in any of these components could impact the system's availability or performance.

``Governance risks``: The system's governance structure could be exploited by malicious actors to seize control or make decisions that harm users.

``Regulatory risks``: The regulatory landscape surrounding cryptocurrency and decentralized finance is evolving rapidly. Changes in regulations could impact the system's legality or operation.

``Smart contract vulnerabilities`` : Smart contracts, like any software, can contain vulnerabilities that could be exploited by attackers. These vulnerabilities could allow attackers to steal funds, manipulate the system, or disrupt operations.

``Circular Dependency ``: A circular dependency occurs when two or more contracts depend on each other in a way that makes it impossible to deploy or upgrade them independently. This can cause problems because the order in which the contracts are deployed or upgraded matters. For example, if the LRTDepositPool contract depends on the NodeDelegator contract, and the NodeDelegator contract depends on the LRTDepositPool contract, then it would be impossible to deploy the NodeDelegator contract without first deploying the LRTDepositPool contract. However, if the LRTDepositPool contract is upgraded after the NodeDelegator contract, then the NodeDelegator contract may not work properly because it is still using the old version of the LRTDepositPool contract.

## Centralization Risks

Many functions in a smart contract heavily dependent on a specific modifier, such as onlyLRTManager.

By restricting certain functionalities to only those with the MANAGER role, there's a centralization of control. This might be contrary to the decentralized and trustless nature that is often a goal in blockchain applications. It can create a single point of failure if the MANAGER role is compromised.

### Recommendations

``Time-Locked Functions``
For critical operations, consider implementing time locks. This means that once an action is initiated, there's a delay before it's executed, allowing time for review or intervention if necessary.

``Role Diversification``
Implement multiple roles for different levels of access and functionalities. This reduces the risk of centralization and allows for more granular control over who can perform specific actions in the contract.

### Privileged Roles
Some privileged roles exercise powers over the Controller contract

#### LRTManager

```solidity

modifier onlyLRTManager() {
        if (!IAccessControl(address(lrtConfig)).hasRole(LRTConstants.MANAGER, msg.sender)) {
            revert ILRTConfig.CallerNotLRTConfigManager();
        }
        _;
    }

```

#### LRTAdmin

```solidity

modifier onlyLRTAdmin() {
        bytes32 DEFAULT_ADMIN_ROLE = 0x00;
        if (!IAccessControl(address(lrtConfig)).hasRole(DEFAULT_ADMIN_ROLE, msg.sender)) {
            revert ILRTConfig.CallerNotLRTConfigAdmin();
        }
        _;
    }

```

#### Minter Role

```solidity

 bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

```
#### Burner Role

```solidity

bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");

```

## Audit Methodology

Project evaluation involves a two-stage process: comprehension and review.

``Comprehension Stage``

During the comprehension stage, the project is examined thoroughly to grasp its background. This includes studying documentation, reviewing previous releases, and analyzing similar protocols. This comprehensive analysis aids in comprehending the project's core value, identifying potential risks, assessing user experiences, and evaluating security. Test coverage is also considered as a crucial factor.

``Review Stage``

The review stage involves a systematic assessment of the project's architecture. Code duplication, inconsistencies, and areas for improvement are thoroughly identified. Best practices are recommended, and optimization strategies are suggested. This comprehensive approach ensures that the project's overall structure and code quality are rigorously scrutinized for potential improvements.

## Time spent:
20 hours






### Time spent:
20 hours