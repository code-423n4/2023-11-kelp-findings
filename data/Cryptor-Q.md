## L-01 No check on whether existing strategy has funds before changing them 

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L109-L122

Protocol does not check whether a strategy has funds in them before changing it. This could present some issues when withdrawing funds



## L-02 No check on whether a strategy has been whitelisted in Eigen Layer 
https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L109-L122
In Eigen Layer's deposit function, there is a check to make sure that a certain strategy is whitelisted. Kelp does not make that same check when it calls Eigen's deposit function. Therefore, there can be situations where a strategy is approved on Kelp but removed from Eigen Layer's whitelist, causing unexpected reverts 



## L-03 Library is un necessary

The Library in this function does not add much utility especially when it come to asset management as the constants are fixed and the admin can add new assets without using the library


## L-04 LRTOracle contracts are not pauseable 

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L31


None of the oracle functions in LRTOracle.sol are pauseable even though the contract inherits pauseable_init. Without it, there could be edge case situations where an outdated oracle is used to fetch incorrect rsETH prices


## L-05 Admin can renounce ownership 

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/3d4c0d5741b131c231e558d7a6213392ab3672a5/contracts/access/AccessControlUpgradeable.sol#L186-L190

Admin should not be able to renounce ownership as it would break several core contract functions