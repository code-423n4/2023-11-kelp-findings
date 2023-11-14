## LRTConfig.sol

### initialize param doc typo
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L40

### updateAssetStrategy() doesn't verify Strategy underlying token
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol#L109-L122

Ignoring Strategy underlying token can lead to errors.

#### Recommendation
Check underlying token
https://github.com/Layr-Labs/eigenlayer-contracts/blob/master/src/contracts/strategies/StrategyBase.sol#L53C19-L53C34

## LRTDepositPool.sol
### There is no option to transfer back assets that have been transferred by mistake.
It's still useful to have such fallback in case of mistakes.
#### Recommendation
Implement admin access function to send assets back.

### transferAssetToNodeDelegator not protected with "whenNotPaused"
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L183-L197 
From overall context it seems that during any error it's better to freeze not only external, but also internal DAO intercommunications

## NodeDelegator.sol

### With current implementation maxApproveToEigenStrategyManager() can be merged with depositAssetIntoStrategy() because there are no another scenarios where approve is needed
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L38-L68


## LRTOracle.sol
### Pricing calculation depends on Eigenlayer shares state
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L52-L79
```
@audit getTotalAssetDeposits will use Eigenlayer Strategy contracts underhood
uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
```
If something bad may happen with Eigenlayer strategy it will have huge impact on the Oracle and Kelp contracts.
#### Recommendation
It make sense to add some additional logic to mitigate the such issues(TWAP, price snapshots, etc).

### No "whenNotPaused" usage but contract is still Pausable
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L19
##### Recommendation 
Verify is Pausable needed for that contract