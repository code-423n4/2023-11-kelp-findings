### Overview of the Protocol

- The protocol intends to create more usage for Liquid Staking Tokens (eg stETH, rETH, cbETH)
- Users can deposit these Liquid Staking Tokens (LST) and get a Liquid Restaking Token (rsETH)
- Users can hold their LRT and use this new token for different purposes, including incurring additional revenue.
- The Liquid Staking Tokens that are deposited in the contract will be manually deposited into Eigen Layer.

### Usage of the Protocol

- Users will call `depositAsset()` in the LRTDepositPool.sol contract to deposit their Liquid Staking Tokens.
- The function will calculate how much Liquid Restaking Tokens to mint, according to the deposits in the Node Delegator contract, the LRT Deposit Pool contract, and the Eigen Layer strategy contract.

### Codebase Quality Analysis and Review

| Emoji                   | Description                            |
| ----------------------- | -------------------------------------- |
| :white_check_mark:      | Fully Checked, Working as Intended     |
| :ballot_box_with_check: | Fully Checked, Requires Slight Changes |
| :red_circle:            | Checked, Something seems off           |

| Contract             | Function                         | Explanation                            | Comments                                                                                                 | Coverage                |
| -------------------- | -------------------------------- | -------------------------------------- | -------------------------------------------------------------------------------------------------------- | ----------------------- |
| LRTConfig            | addNewSupportedAsset             | Adds a new supported asset             | MANAGER, Supported Asset like stEth, rEth, will add according to EigenLayer, there's **no remove asset** | :ballot_box_with_check: |
| LRTConfig            | updateAssetDepositLimit          | Updates the deposit limit for an asset | MANAGER, asset must be supported, deposit limit has no constrains, can be lower than previous limit      | :white_check_mark:      |
| LRTConfig            | updateAssetStrategy              | Updates the strategy for an asset      | ADMIN, asset must be supported, updates the strategy, what's a strategy? - deposit to eigen layer        | :white_check_mark:      |
| LRTConfig            | setToken                         | Sets a token                           | ADMIN, what's a token? - probably a key-value to the strategy                                            | :white_check_mark:      |
| LRTConfig            | setContract                      | Sets a contract                        | ADMIN, what's a contract? Seems like the token and contract goes together                                | :white_check_mark:      |
| LRTDepositPool       | getAssetDistributionData         | Gives Asset amount distribution data   | nodeDelegatorQueue has a max array cap, no out of gas issues                                             | :white_check_mark:      |
| LRTDepositPool       | getRsETHAmountToMint             | Gets rsETH to mint for given asset amt | Works together with LRTOracle getRSETHPrice                                                              | :white_check_mark:      |
| LRTDepositPool       | depositAsset                     | Deposit LST and mint rsETH             | Check deposit limit, transfers LST to the address and mint rsETH                                         | :white_check_mark:      |
| LRTDepositPool       | addNodeDelegatorContractToQueue  | Add new node delegator                 | LRTADMIN, Max 10 node delegators, the **nodedelegatorqueue cannot be popped**                            | :ballot_box_with_check: |
| LRTDepositPool       | transferAssetToNodeDelegator     | Transfer asset to node delegator       | LRTMANAGER, Simple transfer function, success is checked                                                 | :white_check_mark:      |
| LRTDepositPool       | updateMaxNodeDelegatorCount      | Update max node delegator count        | LRTADMIN, Update max delegator from 10, can be more or less                                              | :white_check_mark:      |
| NodeDelegator        | maxApproveToEigenStrategyManager | Max Approve to EigenLayer              | LRTMANAGER, Simple approve function                                                                      | :white_check_mark:      |
| NodeDelegator        | depositAssetIntoStrategy         | Deposit from NDC to strategy           | LRTMANAGER, LST is deposited from LRTDepositPool -> Node Delegator Contract -> Strategy Contract         | :white_check_mark:      |
| NodeDelegator        | transferBackToLRTDepositPool     | Transfer from NDC to LRTDepositPool    | LRTMANAGER, LST is deposited from Node Delegator back to LRTDepositPool                                  | :white_check_mark:      |
| NodeDelegator        | getAssetBalances                 | Get all assets staked in EigenLayer    | view function, get all deposited asset in EigenLayer                                                     | :white_check_mark:      |
| LRTOracle            | getRSETHPrice                    | Provides RSETH/ETH exchange rate       | If no rsETH is minted before, price is 1 ether.                                                          | :red_circle:            |
| LRTOracle            | updatePriceOracleFor             | Add/Update address feed                | Add Address feed for chainlink                                                                           | :white_check_mark:      |
| ChainlinkPriceOracle | getAssetPrice                    | Fetches Asset/ETH exchange rate        | **latestAnswer is deprecated**, check all LST from Chainlink returns 18 decimals                         | :ballot_box_with_check: |
| ChainlinkPriceOracle | updatePriceFeedFor               | Add/Update address feed                | Add Address feed for chainlink                                                                           | :white_check_mark:      |
| RSETH                | mint                             | Mint rsEth                             | Mints rsEth, used in \_mintRsETH in LRTDepositPool, **no burn functionality**                            | :ballot_box_with_check: |
| RSETH                | burnFrom                         | Burns rsEth                            | BURNER, burns rsETH for an account, **can it be frontrunned?**                                           | :red_circle:            |

#### LRTConfig Contract

**LRTConfig Analysis**

- AccessControl is initialized
- All functions have access control, either Manager or admin
- All function has non-zero address check
- Assume that the two access controls are trusted

**LRTConfig Preliminary Questions**

- Supported Assets in `supportedAssetList` array cannot be removed, does this affect any loops? Out of gas calls?
- Deposit limit can be changed. Does this affect anything if it is changed to a smaller number?
- What is a token? What is a contract in `setContract`

**LRTConfig Preliminary Answers**

- It does affect the calculation of rsETH price, because to calculate rsETH, the function has to take into account the prices of all the types of support assets
- Deposit limit is checked when depositing LST in order to mint rsETH. Deposit limit cannot be 0. Otherwise, it is up to the MANAGER role to set a limit. Does not affect any other function.
- Looks like strategy is the EigenStrategyManager contract and other kinds of strategy, from NodeDelegator.sol

#### LRTDepositPool Contract

**LRTDepositPool Analysis**

- Pausable, Reentrancy is initialized
- User can only call `depositAsset()`
- Max 10 Node Delegators, limit can be changed through LRTAdmin

**LRTDepositPool Preliminary Questions**

- There is no burn function? Just mint? So no withdrawing of LST?
- I assume that the tokens are deposited in the proxy? Since this is an upgradeable contract. Seems like users can directly deposit into the implementation contract? Not so sure about this

**LRTDepositPool Preliminary Answers**

- For now, only can mint rsETH and not burn. Once LST is deposited, cannot be redeemed
- I believe they can directly deposit into the implementation contract, but I don't think there's a point, just losing funds for no reason.

### Centralization Risks

| Contract             | Actor      | Description                                             | Severity | Reason                                        |
| -------------------- | ---------- | ------------------------------------------------------- | -------- | --------------------------------------------- |
| LRTConfig            | MANAGER    | Add new supported assets, update asset deposit limit    | High     | Able to set 0 deposit limit to brick protocol |
| LRTConfig            | ADMIN      | Sets the strategy contract and rsETH contract           | Medium   | If set incorrectly, will affect protocol      |
| LRTDepositPool       | LRTManager | Pause contract, transfer assets to node delegator       | Medium   | Transferring of tokens dependent on MANAGER   |
| LRTDepositPool       | LRTAdmin   | Unpause contract, Update Delegator Count                | Medium   | Able to set delegator count                   |
| NodeDelegator        | LRTManager | Pause, Controls transferring of funds to Pool, Strategy | Medium   | Transferring of tokens dependent on MANAGER   |
| NodeDelegator        | LRTAdmin   | Unpause contract                                        | Low      | -                                             |
| LRTOracle            | LRTManager | Pause Contract, update price oracle                     | Medium   | Updates price oracle                          |
| LRTOracle            | LRTAdmin   | Unpause contract                                        | Low      | -                                             |
| ChainlinkPriceOracle | LRTManager | Update price oracle                                     | Medium   | Updates price oracle                          |

**Comments**

- I belive that LRTAdmin has a higher clearance than the LRTManager, so the LRTAdmin should be able to do whatever the LRTManager does as well
- No other external access control given other than protocol
- EigenLayer strategy contract has full approval to use all tokens in NodeDelegator contract.

### Last Comments 

- Relatively small contract, just an extended layer on top of Eigen Layer
- Take note that supported assets cannot be withdrawn
- Take note that Node Delegators cannot be withdrawn

### Time spent:
25 hours