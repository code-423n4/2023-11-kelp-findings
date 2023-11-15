## [1] Use safeMint instead of mint for Protection Against Reentrancy Attacks & overflow/underflow 
https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/RSETH.sol#L48

## [2] No need to initialize ` strategiesLength ` separately since it is only used here in this function ::`getAssetBalances`
https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L105
