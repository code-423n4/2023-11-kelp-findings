## [L-01] COMPILER VERSION TOO RECENT
The compiler version detected in the code is too recent. 
Therefore, it is not time-tested and may be susceptible to multiple bugs and vulnerabilities, both from the usage and security perspectives. 
## Location
`src/oracles/ChainlinkPriceOracle.sol#L2-L2a`
`src/utils/LRTConstants.sol#L2-L2`
`src/LRTConfig.sol#L2-L2`
`src/NodeDelegator.sol#L2-L2`
`src/utils/LRTConfigRoleChecker.sol#L2-L2`
`src/LRTDepositPool.sol#L2-L2`
`src/utils/UtilLib.sol#L2-L2`
`src/RSETH.sol#L2-L2`
`src/LRTOracle.sol#L2-L2`
## Remediation
It is suggested to use a compiler version that is neither too recent nor too old i.e., `Solidity 0.8.18`. 
A stable compiler version should be used that is time-tested by the community, which fixed vulnerabilities introduced in older compiler versions.
The code should be kept updated according to the compiler release cycle. 
It should be tested before going on the Mainnet to reduce the chances of new vulnerabilities being introduced.