If totalETHInPool is smaller than rsEthSupply, this may indicate a potential problem in the contract logic or input data. For example, if the totalETHInPool representing the total amount of ether in the pool is less than rsEthSupply representing the total amount of rsETH (Rari Stable Pool ETH), this may be an incorrect pool state.

``` sol 
if (rsEthSupply >= totalETHInPool) {
    return ..;
} else {
    return totalETHInPool / rsEthSupply; if 1 / 1 = 1; etc or 14/234 = 0,05982(wrong)
}
```


https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L78C15-L78C15

### Time spent:
1 hours