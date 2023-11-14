Outdated pragma solidity version for interface `src/interfaces/IStrategy.sol`

`pragma solidity >=0.5.0;`
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

Do change it to the same version which is `pragma solidity 0.8.21;`