**Kelp DAO**

The Kelp DAO is designed to collectively govern and grow the liquid restaking ecosystem. Key components:

- The DAO treasury will receive fees and rewards from the protocol.

- Kelp token holders can submit and vote on proposals to direct treasury funds.

- Proposals could include adding new asset integrations, funding development, etc.

- The DAO can set parameters like deposit limits and reward rates.

- Smart contracts will execute proposal outcomes. 

**rsETH Token**

The rsETH token represents a claim on assets deposited into the protocol:

- When users deposit supported assets, they receive rsETH in return.

- The total rsETH supply corresponds to the value of assets deposited.

- rsETH can be freely transferred or used in other DeFi protocols.

- Users burn rsETH when they want to withdraw their deposited assets.

- The token is minted/burned by the LRTDepositPool contract.

**Benefits**

- rsETH unlocks liquidity and DeFi for staked ETH and other staked assets.

- Users can earn yields on assets while still earning staking rewards. 

- Governance by Kelp DAO token holders provides decentralized control.

**Architecture**

The architecture centers around the LRTConfig as the main configuration contract that stores addresses of other core contracts like LRTDepositPool and LRTOracle. This provides flexibility to upgrade contracts over time. The admin role in LRTConfig has significant control and permissions. 

Other contracts inherit from base contracts like LRTConfigRoleChecker to enforce admin/manager roles. This is a simple access control scheme without decentralized governance.

**Issue:** Significant centralization risks due to administrative privileges in LRTConfig

**Impact:** The admin role in LRTConfig has permission to update core contract addresses, add/remove assets, update rates, and pause contracts. This provides a central point of control over critical operations.

**Root cause:** 

[LRTConfig.sol](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol):

```solidity
// Admin can set any contract address
function setContract(bytes32 key, address addr) external onlyRole(DEFAULT_ADMIN_ROLE) 

// Admin can add/remove assets
function addAsset(address asset) external onlyRole(DEFAULT_ADMIN_ROLE)

// Admin can pause any contract
function pause(address contractAddr) external onlyRole(DEFAULT_ADMIN_ROLE)
```

**Example attack:**

1. Attacker obtains admin role 
2. Attacker updates LRTDepositPool address to malicious contract
3. User deposits funds into malicious pool contract instead of legitimate one
4. Attacker steals funds

**Recommendations**

- Consider a DAO/governance model where admin privileges are distributed across stakeholders rather than a central admin role.

- External interactions rely heavily on LRTConfig for data. Can optimize gas by storing data together rather than separate calls.

- Use initializer functions rather than constructors for upgradeability best practices.
- - Add escape hatch functions to withdraw funds in case of improper contract updates

**LRTConfig**

This is the main configuration contract that stores addresses of other core contracts and parameters. It uses OpenZeppelin AccessControl to manage admin and manager roles for permissions.

The admin role has significant control - it can set contract addresses, add/remove assets, update parameters like deposit limits, and pause contracts.

Other contracts inherit from LRTConfigRoleChecker to enforce admin/manager permissions.

**LRTDepositPool** 

This is the main user facing contract where funds are deposited. It receives and holds user funds, and mints receipt tokens (RSETH) in return.

It transfers deposited assets to NodeDelegator contracts. It inherits roles from LRTConfigRoleChecker.

**NodeDelegator**

These contracts receive assets from the DepositPool, and delegate the assets into various DeFi protocols for yield. This provides flexibility to plug in different yield strategies.

The NodeDelegators interact with the EigenLayer protocol in the current implementation.

**LRTOracle**

This contract is responsible for getting price data for assets from Chainlink and other oracles. The pricing data is used to calculate RSETH minting amounts when assets are deposited.

**RSETH**

This is an ERC20 token that represents a claim on the deposited assets. It is minted when users deposit, and burned when they withdraw. The total supply correlates to assets locked.

**Flow**

1. Users deposit assets to LRTDepositPool

2. LRTDepositPool transfers assets to NodeDelegators 

3. NodeDelegators provide assets as liquidity to DeFi protocols like EigenLayer 

4. LRTDepositPool mints RSETH tokens to user

**Code Quality**

- Well structured modular code separating key functions.

- Proper use of events for monitoring and OpenZeppelin standards like Ownable.

- Lack of comments in some areas makes following logic difficult.

- Upgradable contracts don't consistently follow initializer pattern.

- No formal specification/documentation of expected behavior.

**Testing**

- Limited unit test coverage of core logic and edge cases.

- Need more scenario based tests, integration tests, and fuzzing.

- Test cases needed for potential manipulation vectors and input validation.

**Potential Issues**

- Centralized control via LRTConfig admin privileges.

- Assets not validated against an approved list before accepting.

- Lack of escape hatches if funds are improperly delegated.

**Assets not validated against approved list**

**Issue**: Assets are not checked against an approved asset whitelist before accepting.

**Impact**: Could allow unapproved or malicious tokens to be deposited, manipulated, or cause issues.

**Code**:

[LRTDepositPool.sol](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol):

```solidity
function deposit(address asset, uint amount) external {

  // No check if asset is approved
  // Vulnerable to depositing unapproved tokens

  balances[asset] += amount; 
}
```

**Recommendation**: Maintain a mapping of approved assets and check before deposit:

```solidity
mapping(address => bool) public approvedAssets; 

function deposit(address asset, uint amount) external {

  require(approvedAssets[asset], "Asset not approved");
  
  // Rest of deposit logic
}
```

**Lack of escape hatches**

**Issue**: No escape hatches if funds are improperly delegated by the admin.

**Impact**: User funds could be trapped if delegated to a faulty contract.

**Code**: 

[LRTConfig.sol](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol)

```solidity
function setDepositPool(address addr) external onlyAdmin {
  depositPool = addr;
}
```

No way to undo improper pool assignment.

**Recommendation**: Add a governance controlled escape hatch: 

```solidity
function withdrawTrappedAssets(address user, address asset) external onlyDAO {
  IAsset(asset).transfer(user, balances[user][asset]) 
}
```



### Time spent:
8 hours