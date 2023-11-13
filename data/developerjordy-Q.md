[L-01] Minting and Burning of `RSETH` is centralized

### Severity

**Impact:** High, as token supply can be endlessly inflated and user tokens can be burned on demand
**Likelihood**: Low, as it requires malicious or compromised minter/burner 

### Description

Currently the `mint` and `burn` methods in `RSETH` are controlled by `MINTER_ROLE` and `BURNER_ROLE` respectively. This means that if the admin or minter or burner account is malicious or compromised it can decide to endlessly inflate the token supply or to burn any user's token balance, which would lead to a loss of funds for users.

[N-01] The `BURNER_ROLE` is never given at deployment.
In the `DeployLRT::setupByAdmin()` the `BURNER_ROLE` is never given to an account. There is a setter for this `AccessControlUpgradeable::grantRole()`

[N-02] Emit event at the end of `NodeDelegator::depositAssetIntoStrategy()`
Best practice is typically to emit events after external function calls or state changes.

[N-03] Missing event after state changes `LRTConfig::updateAssetInStrategy`
Best practice is typically to emit events after external function calls or state changes.

[N-04] Event is missing CamelCase code standard `LRTDepositPool::addNodeDelegatorContractToQueue()` 
The Event `NodeDelegatorAddedinQueue` is missing a CamelCase that is used in their code base `NodeDelegatorAddedInQueue`

[N-05] Inconsistent use of Solidity version
Updatet the Solidity version in `IStrategy` from `0.5.0` to `0.18.21`

