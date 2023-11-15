# DepositLimit can be exceeded due to stETH being a rebase token

## Impact
Although low risk as deposit limit is said to be 100 000 Eth, this will cause problems when interacting with Eigenlayer contracts as they share similar deposit limit.

## Proof of Concept
stETH is a rebasing token that can increase or decrease as the balance of Eth in certain pools changes to maintain 1:1 peg. Should total deposits reach limit specified at any time and stETH supply increases, the protocol will be over the deposit Limit and could encounter problems interacting with the Eignelayer contracts as they are said to have a similar limit.

```solidity
        if (depositAmount > getAssetCurrentLimit(asset)) {
            revert MaximumDepositLimitReached();
        }
```

Likelihood of this happening is low as protocol would have to reach or come really close to deposit limit of 100K ETH first.

## Tools Used
Manuel review

## Recommended Mitigation Steps
Protocol should include separate checks to see if changes in stETH have pushed past depositLimit before depositing into NodeDelegator or EigenLayer strategies. If limit has been passed then protocol should remove some stETH from contract to remain under or equal to depositLimit.