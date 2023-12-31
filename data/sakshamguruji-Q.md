## Make Sure depositLimit is more than totalAssetDeposits(asset) In LRTDepositPool

### Description:

The manager can assign/update the value of depositLimit here https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L102

There should be checks in place to make sure this value is more than totalAssetDeposits in the LRTDepositPool
because here https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L57 it would revert
if deposit limit for the asset was less than the totalAssetDeposits for the asset

## No Withdraw Functionality

### Description:

According to the docs/blogs there should exist a withdraw module which assists rsETH holders in converting their rsETH tokens into a share of all assets managed by the protocol. Redeemers can expect to receive a complex set of various rewards received by Node Delegators for delegating to operators subscribed to various AVSes.

This functionality is either yet to be added or is missing from the logic.

## Add gaps Variable To the Upgradeable Contracts

### Description:

All the kelp contracts are upgradeable.
For upgradeable contracts, there must be storage gap to “allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments” (quote OpenZeppelin). Otherwise it may be very difficult to write new implementation code. Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract. This could have unintended and very serious consequences to the child contracts, potentially causing loss of user fund or cause the contract to malfunction completely.

## Sanity Check To Prevent Rounding Error By Setting A Min Deposit Value

### Description:

We calculate the rsETH to be minted here https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L109
Considering that the price of the asset depegs and is such that amount (considering is very low  let's say 2)
multiplied by `getAssetPrice(asset))` > `getRSETHPrice()` , then the amount to be minted for rsETH would be 
calculated as 0 . 

For recommendation we can introduce a minimum deposit value in the `depositAsset` function


## If The Asset Is An ERC20 Which Does Not Revert On 0 Address Transfer Then Funds Would Be Lost

### Description:

This function https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L183 is used to
transfer assets from the deposit pool to the NDC . 
If the nodeDelegator is a 0 address (mistakenly provided an index in the nodeDelegatorQueue mapping with no value set yet) then funds would be transferred to the 0 address and would be lost forever. A regular ERC20 would revert
in a 0  address transfer but if it is an ERC20 which supports this , then this is an issue.

Eventhough only stETH, rETH, cbETH are supported yet it is possible more diff assets are added in future.


## Part 2: If The Asset Is An ERC20 Which Does Not Revert On 0 Address Transfer Then Funds Would Be Lost

### Description:

Asset can be deposited into the eigen strategy manager from here https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L67 , to get the strategy address we call the `lrtConfig.assetStrategy(asset)` at L59 , but if there's not strategy set yet to the asset this would be a 0 address and the call would revert since we are transferring to 0-address inside Eigen strategy Manager.

If the asset was a token which did not revert on 0 address transfers then funds would have been lost.
Eventhough only stETH, rETH, cbETH are supported yet it is possible more diff assets are added in future.


## 0 Value Sanity Check

### Description:

maxNodeDelegatorCount can be set here by the admin https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L202 ,
There should be a sanity check i.e. `require(maxNodeDelegatorCount!=0)` to make sure there exists delegators
in the system(atleast 1)

## Sanity Check In The `_setContract` Function

### Descriotion:

The _setContract function here https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L172
should have a sanity check to see if the address being added to the mapping (`val`) is indeed a contract , 
this can be done by isContract method which checks if there's any bytecode in the address provided.

## Sanity Check To Ensure That New depositLimit Set Is Not Same As Old

### Description:

Deposit limit for an asset is set here https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L102 . This is also used to update the limit from a previously set value to a 
new one . 
There should be a check to ensure that the new value being set as the limit is not the same as the old deposit limit.
`if(depositLimit[asset] == depositLimit) revert`


## Sanity Check On `_addNewSupportedAsset`

### Description:

New supported Assets can be added from here https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L80
There is a 0 address check for the asset address BUT a similar 0  value check should also be there for the 
depositLimit value.

## Problematic To Be Deployed On L2s

### Descrtiption:

Eventhough currently the protocol is meant to be deployed on Ethereum , it might be possible that the logic focus
shifts to be deployed on L2s like arbitrum too , if that's the case , then since PUSH0 opcode (due to solidity version being 0.8.21) is not supported on such chains it would be problematic to deploy the protocol on those chains.

## Questions Should Be Resolved

### Description:

All the questions and TODOs should be resolved before deployment , there exists a question here https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L78 which should be resolved/removed
before deployment.

## No Need For unchecked Loop Increments

### Description:

The new solidity version i.e. 0.8.22 solidity introduced a remarkable feature that automatically handles overflow checks for loop counters, therefore it would no longer require to have unchecked loop counters which are
proven to be risky.

Bump up the solidity version to 0.8.22

## Sanity check When addNodeDelegatorContractToQueue

### Description:

Node delegators can be added from here https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L170 , there should be a check if the address being added is a contract
or have a interface which should be supported to be a node delegator.

## Introduce A Upper Bound On `nodeDelegatorQueue` To Avoid DoS

### Description:

Node delegators can be added from here https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L170 , there should be an upper bound on how many node delegator
contracts can be added . 


## Name the Array `nodeDelegatorArray` Instead Of `nodeDelegatorQueue`

### Description:

Since `nodeDelegatorQueue` is an array it is better to name it `nodeDelegatorArray` since the properties
of an array and a queue are different . 


## Sanity checks For Balance

### Description:

Here https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L63 there should be a check that
balance is more than 0 , if not then we are transferring 0 value to the strategy manager which would revert.

Also here , https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L86 we are transferring
asset to the deposit pool , make sure the balance of the asset in the NDC is more than 0 so that we dont perform
a 0 value transfer.

## Sanity Check To Ensure Approval Is Done Prior To Depositing Into Eigen

### Description:

NDC can deposit into StrategyManager here https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L51   , for this deposit to happen the NDC should first approve the
strategy manager contract of the assets(max approval) which is done via `maxApproveToEigenStrategyManager` at
L38.
It should be ensured in the `depositAssetIntoStrategy` function before depositing that there is approval to the 
strategy manager or else the call reverts.
A better decision would be to call the `maxApproveToEigenStrategyManager` (declare it public) inside the `depositAssetIntoStrategy`