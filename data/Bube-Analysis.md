The `Kelp Dao` project is a collective DAO designed to unlock liquidity, DeFi and higher rewards for restaked assets through liquid restaking. The contest includes the following smart contracts: `LRTConfig.sol`, `LRTDepositPool.sol`, `NodeDelegator.sol`, `LRTOracle.sol`, `RSETH.sol`, `ChainlinkPriceOracle.sol`, `LRTConfigRoleChecker.sol`, `UtilLib.sol`, `LRTConstants.sol`.

The `LRTConfig.sol` contract is a configuration contract for a system that handle various Ethereum-based assets. It uses `OpenZeppelin's AccessControlUpgradeable` for role-based access control and is designed to be upgradeable. The contract maintains mappings for token addresses, contract addresses, supported assets, deposit limits by asset, and asset strategies. It also keeps a list of supported assets and an address for `rsETH`. The contract's constructor is empty, and the contract is initialized through an `initialize` function, which sets up the initial state of the contract, including the admin role and initial supported assets. The contract includes functions to add new supported assets, update the deposit limit for an asset, update the strategy for an asset, and set the rsETH contract address. These functions are protected by access control, allowing only certain roles to call them. The contract also includes getter functions to retrieve token addresses, contract addresses, and the list of supported assets.

The `LRTDepositPool.sol` contract is responsible for managing deposits of various assets, specifically LST assets, into the protocol. The contract is initialized with a configuration address (`lrtConfigAddr`). The contract also sets a maximum node delegator count (`maxNodeDelegatorCount`) to 10. The contract allows users to deposit assets into the protocol. It checks if the asset is supported, if the deposit amount is non-zero and within the deposit limit. It then transfers the asset from the user to the contract and mints a corresponding amount of rsETH tokens for the user. Also, it provides a function to get the distribution of an asset among the deposit pool, NDCs, and eigenLayer and maintains a queue of node delegator contracts. The LRT admin can add new node delegator contracts to the queue. The LRT manager can transfer assets from the deposit pool to a node delegator contract. The contract can be paused and unpaused by the LRT manager and LRT admin respectively.

The `NodeDelegator.sol` contract is designed to handle the depositing of assets into various strategies. It uses `OpenZeppelin's PausableUpgradeable` and `ReentrancyGuardUpgradeable` contracts to prevent reentrancy attacks and allow pausing of the contract. The contract has several key functions: `initialize` - initializes the contract with a given LRT config address, `maxApproveToEigenStrategyManager` - approves the maximum amount of an asset to the eigen strategy manager. This can only be called by the LRT manager and only for supported assets, `depositAssetIntoStrategy` - deposits an asset into its strategy. This can only be called by the LRT manager and only for supported assets. `transferBackToLRTDepositPool` - transfers an asset back to the LRT deposit pool. This can only be called by the LRT manager and only for supported assets. `getAssetBalances` - fetches the balance of all assets staked in the eigen layer through this contract, `getAssetBalance` - returns the balance of an asset that the node delegator has deposited into the strategy, `pause` and `unpause` - these functions allow the LRT manager to pause the contract and the LRT admin to unpause it.

The `LRTOracle.sol` contract is used to calculate the exchange rate of assets. It imports several libraries and interfaces, including OpenZeppelin's `PausableUpgradeable` contract for pausing functionality and `IERC20` for `ERC20` token standard. The contract has a mapping `assetPriceOracle` that maps an asset address to its price oracle address. It also has a constructor and an initializer function `initialize` which sets the LRT config address. It has two view functions: `getAssetPrice` and `getRSETHPrice`. `getAssetPrice` returns the exchange rate of a given asset by calling the `getAssetPrice` function of the asset's price oracle. `getRSETHPrice` calculates and returns the `RSETH/ETH` exchange rate based on the total supply of RSETH and the total amount of ETH in the deposit pool. The contract also has write functions: `updatePriceOracleFor`, `pause`, and `unpause`. `updatePriceOracleFor` allows the LRTManager to add or update the price oracle of a supported asset. `pause` and `unpause` allow the LRTManager and LRTAdmin to pause and unpause the contract respectively.

The `RSETH.sol` contract is based on the `ERC20` standard, which is a common standard for fungible tokens on the Ethereum blockchain. The contract also includes features for pausing and access control, which are provided by the `OpenZeppelin` library. The contract defines two roles: a minter and a burner. The minter can create new tokens and the burner can destroy tokens. These roles are controlled by the AccessControl feature of the contract. The contract also includes a feature for pausing the contract. When the contract is paused, no new tokens can be minted or burned. The pause function can only be called by the LRT config manager, and the unpause function can only be called by the rsETH admin. It includes a function for updating the LRT config contract. This function can only be called by the rsETH admin.

The `ChainlinkPriceOracle.sol` contract is designed to fetch the exchange rate of assets from Chainlink price feeds. It implements the `IPriceFetcher` interface and uses the `LRTConfigRoleChecker` for role-based access control. The contract also uses `OpenZeppelin's Initializable` contract for upgrade safety. The contract has `assetPriceFeed` a public mapping that links an asset address to its corresponding Chainlink price feed address. The `constructor` is empty and calls `_disableInitializers()` to prevent the initializer functions from being called more than once. The `initialize` function sets the LRT config address after checking that it's not a zero address. It emits an event `UpdatedLRTConfig`. The `getAssetPrice` function fetches the latest exchange rate for a given asset from its corresponding Chainlink price feed. It can only be called for supported assets. The `updatePriceFeedFor` function allows the LRTManager to add or update the price feed for a supported asset. It checks that the new price feed address is not a zero address before updating the mapping and emitting an `AssetPriceFeedUpdate` event.

The `LRTConfigRoleChecker.sol`contract is used to manage roles and permissions in a system. It imports several libraries and interfaces, and uses OpenZeppelin's `IAccessControl` for role-based access control. The contract has three modifiers: `onlyLRTManager`- this modifier ensures that the function it is attached to can only be called by an account that has the `MANAGER` role in the `lrtConfig` contract, `onlyLRTAdmin`- ensures that the function it is attached to can only be called by an account that has the `DEFAULT_ADMIN_ROLE` in the `lrtConfig` contract and `onlySupportedAsset` - ensures that the function it is attached to can only be called with an asset address that is supported according to the `lrtConfig` contract. The contract also has a function `updateLRTConfig` that allows an account with the `LRTAdmin` role to update the `lrtConfig` contract address. The function checks that the new address is not zero, updates the `lrtConfig` state variable, and emits an `UpdatedLRTConfig` event.

The `UtilLib.sol` contract defines a library named `UtilLib` that provides utility functions. The library has a single function `checkNonZeroAddress(address address_)` which checks if the provided address is a zero address (0x0000000000000000000000000000000000000000). If the address is a zero address, it reverts the transaction and throws a custom error `ZeroAddressNotAllowed()`. This function is useful to prevent contracts from interacting with the zero address, which is a common security precaution in Ethereum smart contracts. The zero address is often used as a default value and sending funds to it would effectively burn them, making them unrecoverable.

The last `LRTConstants.sol` contract contains a set of constant values. These constants are hashed representations of string literals, which are used as identifiers for tokens, contracts, and roles within a larger system. The tokens section includes `R_ETH_TOKEN`, `ST_ETH_TOKEN`, and `CB_ETH_TOKEN`, which likely represent different types of Ethereum tokens within the system. The contracts section includes `LRT_ORACLE`, `LRT_DEPOSIT_POOL`, and `EIGEN_STRATEGY_MANAGER`, which likely represent different contracts within the system. The roles section includes `MANAGER`, which likely represents a role within the system.

The whole project is well written and structured. It was interesting to work on it.

### Time spent:
15 hours