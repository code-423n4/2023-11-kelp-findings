###### An attacker can grief users  with large amounts of funds 
An attacker can grief hard by depositing funds before a call is made making the  
example:
price = 100 eth = 50 rseth 
attacker deposits 25 eth 
price =100 eth = 35 rseth 
There is no slippage check so it can harm users.
```solidity
 rsethAmountToMint = (amount * lrtOracle.getAssetPrice(asset)) / lrtOracle.getRSETHPrice();
```
###### If there are some delegators that are not contracts it will revert 
The sponsor says that there will be delegators to transfer the funds to the strategy.
If an admin adds an eoa  to the [delegators array](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L81C5-L84C101) it will revert
```solidity
    uint256 ndcsCount = nodeDelegatorQueue.length;
        for (uint256 i; i < ndcsCount;) {
            assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
```
###### Their can be an issue with a user transferring x token but getting the price for x,y,b tokens 
Since the rsETh price is computed using all the supported assets, if some of them are overpriced or underpriced it will affect how many eth are in the contract which can cause txs to fail when the price rises or when it lowers more funds can be deposited.
###### comment is wrong the function is called by Admin not the [manager](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L160)
```solidity
/// @notice add new node delegator contract addresses/// @dev only callable by LRT manager/// @param nodeDelegatorContracts Array of NodeDelegator contract addresses function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {
```
###### Their is no check for the heartbeat of the oracle which can return stale prices 
Most of the supported tokens, update every [12/13 hours](https://data.chain.link/ethereum/mainnet/crypto-eth/cbeth-eth)
So the price can be stale 
###### If the strat that the manager is going to deposit to has tvl limit an attacker can front run and make the call revert 
https://github.com/Layr-Labs/eigenlayer-contracts/blob/db4506d07b2b9029c31d583d5da6b790484c2b95/src/contracts/strategies/StrategyBaseTVLLimits.sol#L79
###### [eigenlayer owner](https://github.com/Layr-Labs/eigenlayer-contracts/blob/db4506d07b2b9029c31d583d5da6b790484c2b95/src/contracts/core/StrategyManager.sol#L296) can frontrun the manager tx and cause it revert in `depositAssetIntoStrategy`
the eigenlayer  owner  can make that strat whitelisted and then not whitelisted again causing a revert 
###### an attacker can make [getassetbalances](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L102) consume more gas 
If an attacker wants they can supply wei to alot of strategies and fill up the strategies array consuming more gas
###### since [getassetbalances](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L102) will return all strategies that a user has deposited onbehlaf of depositPool it can return a wrong token
For example, if a strategy  was listed on eignLayer but not in the depositPool the function will show that that token is their
```solidity
assets[i] = address(IStrategy(strategies[i]).underlyingToken());
```
In the future, there can be unlimited strategies and tokens can be added so it can allow a phishing token to be added on the frontend/function
###### if one of the contracts is paused, all contracts should be paused 
ex: 
rsETH is paused then the whole system should be paused and all calls should revert you shouldn't be able to remove rsETH token to new one and then be able to deposit it. 
You should also implement the same state so you can't have state where 
Admin -> pauses token contract 
Admin sets new token contract 
user can deposit maybe when the admin didn't want that action.
Admin should pause the whole state so they can manage the whole state.