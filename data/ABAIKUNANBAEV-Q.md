# Low-level issues

## Finding Summary 

| ID | Description | Severity |
| - | - | :-: |
| [L-01](#l-01-relience-on-total-supply) | Reliance on `totalSupply()` in `getRSETHPrice()` makes it susceptible to front-running attacks and manipulations | Low |


## [L-01] Reliance on `totalSupply()` in `getRSETHPrice()` makes it susceptible to front-running attacks and manipulations

Inside of `LRTOracle.sol` the price of rsETH is determined by `totalETHInPool and `rsEthSupply`:

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L78
```
return totalETHInPool / rsEthSupply;

```

The problem is that the `rsEthSupply` can be easily manipulated and, therefore, the `getRsETHAmountToMint()` will be influenced as well:

https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L109

```
rsethAmountToMint = (amount * lrtOracle.getAssetPrice(asset)) / lrtOracle.getRSETHPrice();
```

Possible mitigation: consider using already existing oracles instead of implementing the new one.