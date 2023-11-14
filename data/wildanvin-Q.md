## Use of `override` is unnecessary

As mentioned by the bot report: Starting with Solidity version [0.8.8](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding), using the override keyword when the function solely overrides an interface function, and the function doesn't exist in multiple base contracts, is unnecessary.

The report mentions that exists 13 instances of this issue but it missed the one at:

```solidity
File: src/LRTOracle.sol

20: 		        mapping(address asset => address priceOracle) public override assetPriceOracle;
```

[[20](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L20)]