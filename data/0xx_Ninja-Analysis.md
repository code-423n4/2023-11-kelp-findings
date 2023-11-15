### Architecture
The Stader Labs protocol comprises several key contracts, each serving a specific role in the ecosystem:
1.LRTConfig:

Responsible for managing configurations, including supported assets, deposit limits, and asset strategies.
Maintains mappings for token and contract addresses.

2.RSETH:

An upgradeable ERC20 token contract named "rsETH."
Manages minting and burning of tokens, controlled by specific roles.
Implements pausing and unpausing functionality.

3.LRTDepositPool:

Handles the deposit of assets from users, forwarding funds to NodeDelegator contracts.
Tracks asset distribution across the deposit pool, NodeDelegators, and EigenLayer protocol.
Manages the minting of rsETH tokens based on deposited assets.

4.NodeDelegator:

Contracts responsible for receiving funds from LRTDepositPool and delegating them to the EigenLayer strategy.
Supports actions such as depositing assets into the strategy and transferring assets back to LRTDepositPool.
Implements pausing and unpausing functionality.

5.LRTOracle:

Fetches and provides asset prices, as well as the rsETH price.
Uses oracles and data from the deposit pool to determine rsETH price.

### Centralization
The protocol exhibits a degree of centralization in the following aspects:

Admin Roles:

Various contracts utilize role-based access control, with roles like `DEFAULT_ADMIN_ROLE` and custom roles such as `MINTER_ROLE` and `BURNER_ROLE`.
Centralized control over key functions is concentrated in the hands of administrators.
Initialization:

The contracts use initializers to set up roles and configurations, controlled by administrators.

### Systematic Risk
Several systematic risks should be considered:

**Upgradeability:**

The protocol employs upgradeable contracts, introducing the risk of unintended changes or vulnerabilities in future upgrades.
**Oracle Dependency:**

Asset prices and the rsETH price rely on oracles, introducing potential vulnerabilities if the oracle infrastructure is compromised.

**Role-based Access Control:**

Centralized roles control critical functions, and a compromised admin could pose risks to the protocol.

**Pausing Mechanism:**

The ability to pause and unpause certain functions introduces operational risk, and misuse of these controls could impact user access.

**Asset Strategies:**

The effectiveness of the protocol depends on the strategies employed for asset management, and vulnerabilities in these strategies could lead to financial risks.

### Time spent:
13 hours