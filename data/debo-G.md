## [G-01] DEFINE CONSTRUCTOR AS PAYABLE
## Impact
Developers can save around 10 opcodes and some gas if the constructors are defined as payable.
However, it should be noted that it comes with risks because payable constructors can accept ETH during deployment.
## Location
`src/oracles/ChainlinkPriceOracle.sol#L21-L23`
## Remediation
It is suggested to mark the constructors as payable to save some gas. Make sure it does not lead to any adverse effects in case an upgrade pattern is involved.