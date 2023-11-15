# Overall Analysis
* ## About the Protocol
  > [KelpDAO](https://twitter.com/KelpDAO) is a collective DAO designed to unlock liquidity, DeFi and higher rewards for restaked assets.

</br>

* ## Low Severity Issues
  * **[[L-01] Prevent deposits to the nodes with max amount of assets being staked already](#l-01-prevent-deposits-to-the-nodes-with-max-amount-of-assets-being-staked-already)**
  * **[[L-02] Set the price feeds during an initialization](#l-02-set-the-price-feeds-during-an-initialization)**
  * **[[L-03] Dropping a dust to grief the staking process](#l-03-dropping-a-dust-to-grief-the-staking-process)**
  * **[[L-04] Approve to `StrategyManager` during an initialization](#l-04-approve-to-strategymanager-during-an-initialization)**
  * **[[L-05] Inforce `maxnodeDelegatorCount > nodeDelegatorQueue.length`](#l-05-inforce-maxnodedelegatorcount--nodedelegatorqueuelength)**
  * **[[L-06] Place an assertion to ensure the system in paused/unpaused state](#l-06-place-an-assertion-to-ensure-the-system-in-pausedunpaused-state)**

</br>

* ## Non-Critical Severity Issues
  * **[[N-01] There is no need to guard with `nonReentrant`](#n-01-there-is-no-need-to-guard-with-nonreentrant)**
  * **[[N-02] Introduce `removeNodeDelegatorToQueue`](#n-02-introduce-removenodedelegatorfromqueue)**

</br>

## **[L-01] Prevent deposits to the nodes with max amount of assets being staked already**<a name="L-01"></a>

- If the node has staked already 1e23 of assets, it should not receive any transfers, since there is no more space to stake at EigenLayer.

### **Example of an occurance:**

- Could be implied [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L183-L197):


</br>

## **[L-02] Set the price feeds during an initialization**<a name="L-02"></a>

- Currently, the price feeds are setted by using `updatePriceOracleFor()` after an `LRTOPracle` being deployed. However, it's better to set up everything during an initialization process.

### **Example of an occurance:**

- Could be added, for instances, [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L29-L35):


</br>

## **[L-03] Dropping a dust to grief the staking process**<a name="L-03"></a>
- After an amount of lst being transfered to a specified `NodeDelegator`, the stake on EigenLayer is expected to happen. However, if there is a possibility to grief `depositAssetIntoStrategy`, adversary might drop some dust and front-run the tx submitted by `LRTManager`. This forces to transfer some assets back to the pool in order to successfully stake at EigenLayer.
  
### **Example of an occurance:**

- It's better to provide an opportunity for `LRTManager` to decide, how much lst amount is about to be transfered and not blindly rely on the total balance [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L63):

</br>

## **[L-04] Approve to `StrategyManager` during an initialization**<a name="L-04"></a>
- There is no need to inflate the bytecode for `NodeDelegator` by introducing a special function for an approval grants. Instead, approve to the `eigenlayerStrategyManagerAddress` during an initialization.
  
### **Example of an occurance:**

-  Could be added, for instances, [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L29-L35):

</br>

## **[L-05] Inforce `maxnodeDelegatorCount > nodeDelegatorQueue.length`**<a name="L-05"></a>
- Put an assertion to ensure that the `maxnodeDelegatorCount > nodeDelegatorQueue.length` to prevent unexpected behavior from happening. 
  
### **Example of an occurance:**

-  Could be added, for instances, [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L202-L205):

</br>

## **[L-06] Place an assertion to ensure the system in paused/unpaused state**<a name="L-06"></a>
- The functions `pause()/unpause()` are super important in a case, when something suspicous start to happen. An `LRTManager` might accidentally invoke `unpause()` instead of hitting `pause()` and convince himself of stopping suspicous activity. However, this might give an attacker the chance to continue his dirty activity.
  
### **Example of an occurance:**

-  Could be added, for instances, [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L208-L215):

</br>

## **[N-01] There is no need to guard with `nonReentrant`**<a name="N-01"></a>
- Some of the functions are overprotected by `nonReentrant` modifier, however, the re-entrancy is not possible. It causes an unnecessary gas overhead. 
  

### **Example of an occurance:**

-  Could be removed, for instances,
   -   [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L189):
   -   [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L125):
   -   [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L55):
   -   [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L80):


</br>

## **[N-02] Introduce `removeNodeDelegatorFromQueue`**<a name="N-02"></a>
- It is recommended to introduce `removeNodeDelegatorFromQueue` in case if the `NodeDelegator` is not longer needed.
  

### **Example of an occurance:**

-  Could be introduced, for instances, [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L158):

</br>