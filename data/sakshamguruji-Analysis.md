# Analysis Of Kelp Protocol

## Codebase Quality:

The codebase is well formatted and demonstrates exceptional quality . It is clear that devs have written this protocol with security
being the priority.

To make the codebase readable and understandable natspec have been utilized.

There are unit tests written in foundry , but , the coverage is not up to the mark. Several branches have not been taken care of
and are below 80% covered in some cases . This should be improved .

Moreover there should be an addition of fuzz and invariant testing.

There was no formal documentation provided except a blog post , there should have been proper documentation provided to the auditor's
to make the audit process seamless and quick.

## Mechanism Review

### Architecture And Working:


![Graph1](https://user-images.githubusercontent.com/93149832/282935945-44e6cb81-70ad-4fa8-bf0a-f0bf3f667feb.png)

This is a high level overview of the system , it was intentional to not complicate the graph with every function in each contract.
The LRTConfig.sol has not been depicted since it's funcction is to just update/assign values to state variables.


#### _LRTDepositPool.sol_

* _Upgradable_ : Yes

* _Pause Functionality_  : Yes (Manager can pause and Admin can unpause)

Contract which lets the users deposit their asset (should be a supported asset) into the contract via the
`depositAsset` function . Depending upon the deposited amount the depositor is minted rsETH tokens (yield bearing token).

_interactions_ : To fetch the price of the asset and to fetch the price of the rsETH (in the depositAsset function) the contract
calls LRTOracle which calls the PriceFetcher contract to get the asset's price.

The assets that are deposited into the pool are then distributed/transferred to the NDC (the NodeDelegator.sol contract) via 
the `transferAssetToNodeDelegator` function which can only be called by the manager role.
To choose which nodeDelegator to send the funds to an index is provided in the function call which is used to lookup the 
node delegator address inside the `nodeDelegatorQueue` array.

Only the manager can add node delegators to the `nodeDelegatorQueue` array using the `addNodeDelegatorContractToQueue` function.

#### _NodeDelegator.sol_

* _Upgradeable_ : Yes
 
* _Pause Functionality_ : Yes (Manager can pause and Admin can unpause)

As explained above , the deposit pool deposits assets into the NodeDelegator contract . These assets are then deposited into the
EigenLayerStrategyManager contract via the `depositAssetIntoStrategy` function . 

_interactions_ : The function `depositAssetIntoStrategy` calls the `depositIntoStrategy` function of the EigenLayerStrategyManager
contract , it takes strategy (which is looked up via the `assetStrategy` mapping inside LRTConfig.sol) , token (the asset) and the balance 
of the token inside the contract (the node delegator contract) as input parameters.

It is not mandatory to deposit the assets into the EigenLayerStrategyManager , the `manager` can also call the `transferBackToLRTDepositPool`
which would transfer the assets back into the deposit pool.

To know which asset and how much of that asset has been deposited into the Eigen Layer , `getAssetBalances` should be called.

#### _LRTConfig.sol_

* _Upgradeable_ : Yes
 
* _Pause Functionality_ : No

This is the configuration heart of the kelp protocol. 
Configurations taken care of ->

* Adding a new supported asset (synonymous to a whitelist) 
* Assigning/Updating the asset's (should be supported) deposit limit.
* Assigning/Updating the startegy to an asset . (Done via updating the `assetStrategy` mapping for the asset)
* Assigning/Updating the rsETH token address.
* Assigning/Updating tokenMap/contractMap mappings (mappings which map a bytes32 key value to a token address)

All the above can be performed by the manager or the admin (adding a new supported asset -> manager , rest callable by only the default admin)

#### _RSETH.sol_

* _Upgradeable_ : Yes
 
* _Pause Functionality_ : Yes (Manager can pause and Admin can unpause)

This is an  ERC20 token contract representing the rsETH token. Like we discussed above  , this is the token wihch is minted to the 
asset depositor in the LRTDepositPool contract

The minting and burning of this token is rstricted to the MINTER_ROLE and the BURNER_ROLE respectively.

#### _LRTOracle.sol_

* _Upgradeable_ : Yes
 
* _Pause Functionality_ : Yes (Manager can pause and Admin can unpause)

This is the contract which is used to fetch the price of the asset (including the rsETH price calculation). Prices are denominated in ETH
i.e. prices are in the form of Asset/ETH

_interations_ : To fetch the price `getAssetPrice` function is called (should be supported asset) which in turn calls the PriceFetcher's (an OOS interaction)
`getAssetPrice(asset)`  function to get the price from the oracle.

The minting of rsETH (how much to mint) is dependent on the rsETH price fetched which is calculated as the average price of
all supported assets inside the `getRSETHPrice` function.

To know which price oracle to query , the `assetPriceOracle` mapping is used to lookup the oracle address from the asset. This
mapping can be updated by the manager via the `updatePriceOracleFor` function.


## Architecture Recommendations

The system architecture and logic is pretty simple and straightforward and the kelp dev team has done a fantastic job in laying out
everything in a very understandable and "secure" way. These are some of the architecture recommendations that I would like to talk
about that might help in making the system more robust and user-friendly ->

* Right now assets can be deposited via the Deposit Pool contract and rsETH is minted , there should be minimum requirements for such
actions , in some edge cases it might be possible that the user deposits some little "x" value of the asset and gets some dust amount or 0 rsETH in return.

* The contracts are upgradeable with inheritance,  there must be storage gap to allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments . Otherwise it may be very difficult to write new implementation code. Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract.

* Have a whitelist of assets that can be added into the supported assets , even though the process is trusted this should atleast
be documented.

* The flow in the NDC to deposit into the Strategy Manager is , maxApproveToEigenStrategyManager -> depositAssetIntoStrategy,
to make the flow more streamline maxApproveToEigenStrategyManager should be called inside depositAssetIntoStrategy.

* It is recommended to have proper natspec for state variables and struct too so that it is easier to comprehend the system logic and intention.




## Centralisation Risks

The kelp protocol has handled the "centralised problem" pretty well by assigning different roles to different functionalities.
The LRTManager can pause contracts but they can be unpaused by a different role i.e. LRTAdmin , this would ensure that
eventhough the LRTManager is compromised/malicious and wants to pause the system forever it can be undone via a different address/rols.

Eventhough there are roles for different functionalities such as ->

* MINTER_ROLE 
* BURNER_ROLE
* LRTManager
* LRTAdmin
* DEFAULT_ADMIN

, there still exists centralisation risks . The DEFAULT_ADMIN can update system critical values such as the address of the rsETH
token , the supported assets and set tokens and contracts . 
It is essential to make sure that these privileges addresses are a multisig with a timelock , moreover each key should reside on
a different server so that compromise of 1 server does not lead to compromise of all the keys.


## Systemic risks

1.) To fetch the price of the asset (including price of rsETH which is calculated based on the amounts of other assets) chainlink price
feeds are used , if the feeds go down or malfunction this would destabalize the system . The use of `latestAnswer()` makes this
argument stronger since it is a deprecated function and does not prevent the oracle from giving stale prices.

2.) The assets deposited in the deposit pool and sent to the node delegator which delegates the funds to the Eigen Layer Strategy Manager , if the eigen layer protocol gets compromised and is paused then the deposit functionality would be paused too . This would 
mean that funds can not be deposited to the eigen contract anymore and the kelp system would be paused too.

3.) The contract are written in solidity version 0.8.21 , even though the main intention is to deploy the protocol on mainnet it should 
be noted that it might be problematic to deploy them on L2's due to the PUSH0 opcode not being supported by L2 chains.

4.) Governance risks  - According to the blog : "The reward market offers various strategies for engaging different rewards. These decisions are driven by the DAO through governance" , there might be cases where a malicious strategy is proposed which might lead
to users loosing their funds.

5.) The depositors (depositing into the deposit pool) are dependent on the node delegators to deposit their LST into EigenLayer.


## Key Takeaways

* The overall quality of the codebase is excellent and it is evident that the protocol was written keeping security as the priority.
* Kelp has brought forward the new innovative idea , Liquid Restaking Token . Users can stake their ETH and support validation on multiple networks (This is done through leveraging the Eigen Layer Protocol)
* The contracts are upgradeable and logic can added , for example , the withdraw functionality is yet to be added into the system
which is currently not present (though this addition is not related to upgradeability this is to show how extra logic might need to be 
added in the future)

## Time Spent 

10 hours



### Time spent:
10 hours