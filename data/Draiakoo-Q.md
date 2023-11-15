## Title
The following invariant can be broken:
maxNodeDelegatorCount >= nodeDelegatorQueue.length

## Explanation
The protocol should hold the previous exposed invariant, however it is possible to make it maxNodeDelegatorCount < nodeDelegatorQueue.length by setting a limit lower than the current number of nodes.

## Proof of Concept
```
// SPDX-License-Identifier: UNLICENSED

pragma solidity 0.8.21;

import { Test } from "forge-std/Test.sol";
import "forge-std/console.sol";
import { ERC20 } from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import { TransparentUpgradeableProxy } from "@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol";
import { ProxyAdmin } from "@openzeppelin/contracts/proxy/transparent/ProxyAdmin.sol";
import { LRTConfig, ILRTConfig } from "src/LRTConfig.sol";
import { RSETH } from "src/RSETH.sol";
import { LRTDepositPool } from "src/LRTDepositPool.sol";

import { LRTConstants } from "src/utils/LRTConstants.sol";
import { PausableUpgradeable } from "@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol";

contract MockToken is ERC20 {
    constructor(string memory name, string memory symbol) ERC20(name, symbol) { }

    function mint(address to, uint256 amount) external {
        _mint(to, amount);
    }
}

contract LRTOracleMock {

    LRTConfig public lrtConfig;
    mapping(address asset => uint256 price) public prices;

    constructor(address lrtConfigAddr) {
        lrtConfig = LRTConfig(lrtConfigAddr);
    }

    function setAssetPrice(address asset, uint256 amount) external {
        prices[asset] = amount;
    }

    function getAssetPrice(address asset) public view returns (uint256) {
        return prices[asset];
    }

    function getRSETHPrice() external view returns (uint256 rsETHPrice) {
        address rsETHTokenAddress = lrtConfig.rsETH();
        uint256 rsEthSupply = RSETH(rsETHTokenAddress).totalSupply();

        if (rsEthSupply == 0) {
            return 1 ether;
        }

        uint256 totalETHInPool;
        address lrtDepositPoolAddr = lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL);

        address[] memory supportedAssets = lrtConfig.getSupportedAssetList();
        uint256 supportedAssetCount = supportedAssets.length;

        for (uint16 asset_idx; asset_idx < supportedAssetCount;) {
            address asset = supportedAssets[asset_idx];
            uint256 assetER = getAssetPrice(asset);

            uint256 totalAssetAmt = LRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
            totalETHInPool += totalAssetAmt * assetER;

            unchecked {
                ++asset_idx;
            }
        }

        return totalETHInPool / rsEthSupply;
    }
}

contract MockNodeDelegator {
    function getAssetBalance(address) external pure returns (uint256) {
        return 1e18;
    }
}

contract BaseTest is Test {
    MockToken public stETH;
    MockToken public rETH;
    MockToken public cbETH;

    address public admin = makeAddr("admin");

    address public alice = makeAddr("alice");
    address public bob = makeAddr("bob");
    address public carol = makeAddr("carol");

    uint256 public oneThousand = 1000 ** 18;

    function setUp() public virtual {
        stETH = new MockToken("staked ETH", "stETH");
        rETH = new MockToken("rETH", "rETH");
        cbETH = new MockToken("cbETH", "cbETH");

        // mint LST tokens to alice, bob and carol
        mintLSTTokensForUsers(stETH);
        mintLSTTokensForUsers(rETH);
        mintLSTTokensForUsers(cbETH);
    }

    function mintLSTTokensForUsers(MockToken asset) internal {
        asset.mint(alice, oneThousand);
        asset.mint(bob, oneThousand);
        asset.mint(carol, oneThousand);
    }

    /// @dev Expects an event to be emitted by checking all three topics and the data. As mentioned
    /// in the Foundry
    /// Book, the extra `true` arguments don't hurt.
    function expectEmit() internal {
        vm.expectEmit({ checkTopic1: true, checkTopic2: true, checkTopic3: true, checkData: true });
    }
}

contract AuditTests is Test {
    MockToken public stETH;
    MockToken public rETH;
    MockToken public cbETH;
    LRTConfig public lrtConfig;
    RSETH public rseth;
    LRTDepositPool public lrtDepositPool;
    LRTOracleMock public lrtOracle;
    address public manager = makeAddr("manager");

    address public admin = makeAddr("admin");

    address public alice = makeAddr("alice");
    address public bob = makeAddr("bob");
    address public carol = makeAddr("carol");
    address public randomUser = makeAddr("randomUser");

    function setUp() public virtual {
        stETH = new MockToken("staked ETH", "stETH");
        rETH = new MockToken("rETH", "rETH");
        cbETH = new MockToken("cbETH", "cbETH");

        ProxyAdmin proxyAdmin = new ProxyAdmin();
        LRTConfig lrtConfigImpl = new LRTConfig();
        TransparentUpgradeableProxy lrtConfigProxy = new TransparentUpgradeableProxy(
            address(lrtConfigImpl),
            address(proxyAdmin),
            ""
        );

        RSETH tokenImpl = new RSETH();
        TransparentUpgradeableProxy tokenProxy = new TransparentUpgradeableProxy(
            address(tokenImpl),
            address(proxyAdmin),
            ""
        );

        LRTDepositPool contractImpl = new LRTDepositPool();
        TransparentUpgradeableProxy contractProxy = new TransparentUpgradeableProxy(
            address(contractImpl),
            address(proxyAdmin),
            ""
        );

        lrtConfig = LRTConfig(address(lrtConfigProxy));
        lrtConfig.initialize(admin, address(stETH), address(rETH), address(cbETH), address(1));

        lrtOracle = new LRTOracleMock(address(lrtConfig));

        lrtDepositPool = LRTDepositPool(address(contractProxy));
        lrtDepositPool.initialize(address(lrtConfig));

        rseth = RSETH(address(tokenProxy));
        rseth.initialize(address(admin), address(lrtConfig));

        vm.startPrank(admin);
        lrtConfig.grantRole(LRTConstants.MANAGER, manager);
        lrtConfig.setContract(LRTConstants.LRT_ORACLE, address(lrtOracle));
        lrtConfig.setContract(LRTConstants.LRT_DEPOSIT_POOL, address(lrtDepositPool));
        lrtConfig.setRSETH(address(rseth));
        rseth.grantRole(rseth.MINTER_ROLE(), address(lrtDepositPool));
        vm.stopPrank();

        // set prices for tokens
        lrtOracle.setAssetPrice(address(stETH), 1 ether);       // 1 ETH    = 1 stETH
        lrtOracle.setAssetPrice(address(rETH), 1.05 ether);     // 1.05 ETH = 1 rETH
        lrtOracle.setAssetPrice(address(cbETH), 1.1 ether);     // 1.1 ETH  = 1 cbETH
    }

    function test_NodeDelegatorLimitCanBeSetToLessThanCurrentAmountOfThem() public {
        vm.startPrank(admin);
        MockNodeDelegator delegator1 = new MockNodeDelegator();
        MockNodeDelegator delegator2 = new MockNodeDelegator();
        MockNodeDelegator delegator3 = new MockNodeDelegator();
        MockNodeDelegator delegator4 = new MockNodeDelegator();
        MockNodeDelegator delegator5 = new MockNodeDelegator();
        address[] memory newNodes = new address[](5);
        newNodes[0] = address(delegator1);
        newNodes[1] = address(delegator2);
        newNodes[2] = address(delegator3);
        newNodes[3] = address(delegator4);
        newNodes[4] = address(delegator5);
        lrtDepositPool.addNodeDelegatorContractToQueue(newNodes);

        // it is possible to change node delegator limit to less than the current amount of them
        lrtDepositPool.updateMaxNodeDelegatorCount(2);

        address[] memory nodeDelegatorsQueue = lrtDepositPool.getNodeDelegatorQueue();
        assertGt(nodeDelegatorsQueue.length, lrtDepositPool.maxNodeDelegatorCount());
    }
}
```
Traces
```
Running 1 test for test/BaseTest.t.sol:AuditTests
[PASS] test_NodeDelegatorLimitCanBeSetToLessThanCurrentAmountOfThem() (gas: 546943)
Traces:
  [546943] AuditTests::test_NodeDelegatorLimitCanBeSetToLessThanCurrentAmountOfThem()
    ├─ [0] VM::startPrank(admin: [0xaA10a84CE7d9AE517a52c6d5cA153b369Af99ecF])
    │   └─ ← ()
    ├─ [38891] → new MockNodeDelegator@0x3Ede3eCa2a72B3aeCC820E955B36f38437D01395
    │   └─ ← 194 bytes of code
    ├─ [38891] → new MockNodeDelegator@0x6D9da78B6A5BEdcA287AA5d49613bA36b90c15C4
    │   └─ ← 194 bytes of code
    ├─ [38891] → new MockNodeDelegator@0xDDd9A038D57372934f1b9c52bd8621F5ED4268DF
    │   └─ ← 194 bytes of code
    ├─ [38891] → new MockNodeDelegator@0x32850cAd1e9170614704fF8BA37a25e498e1B832
    │   └─ ← 194 bytes of code
    ├─ [38891] → new MockNodeDelegator@0x1742F7d30605F18f097F356E1EeAfDB5e2FDe33a
    │   └─ ← 194 bytes of code
    ├─ [167505] TransparentUpgradeableProxy::addNodeDelegatorContractToQueue([0x3Ede3eCa2a72B3aeCC820E955B36f38437D01395, 0x6D9da78B6A5BEdcA287AA5d49613bA36b90c15C4, 0xDDd9A038D57372934f1b9c52bd8621F5ED4268DF, 0x32850cAd1e9170614704fF8BA37a25e498e1B832, 0x1742F7d30605F18f097F356E1EeAfDB5e2FDe33a])
    │   ├─ [160380] LRTDepositPool::addNodeDelegatorContractToQueue([0x3Ede3eCa2a72B3aeCC820E955B36f38437D01395, 0x6D9da78B6A5BEdcA287AA5d49613bA36b90c15C4, 0xDDd9A038D57372934f1b9c52bd8621F5ED4268DF, 0x32850cAd1e9170614704fF8BA37a25e498e1B832, 0x1742F7d30605F18f097F356E1EeAfDB5e2FDe33a]) [delegatecall]
    │   │   ├─ [9792] TransparentUpgradeableProxy::hasRole(0x0000000000000000000000000000000000000000000000000000000000000000, admin: [0xaA10a84CE7d9AE517a52c6d5cA153b369Af99ecF]) [staticcall]
    │   │   │   ├─ [2694] LRTConfig::hasRole(0x0000000000000000000000000000000000000000000000000000000000000000, admin: [0xaA10a84CE7d9AE517a52c6d5cA153b369Af99ecF]) [delegatecall]
    │   │   │   │   └─ ← true
    │   │   │   └─ ← true
    │   │   ├─ emit NodeDelegatorAddedinQueue(prospectiveNodeDelegatorContract: MockNodeDelegator: [0x3Ede3eCa2a72B3aeCC820E955B36f38437D01395])
    │   │   ├─ emit NodeDelegatorAddedinQueue(prospectiveNodeDelegatorContract: MockNodeDelegator: [0x6D9da78B6A5BEdcA287AA5d49613bA36b90c15C4])
    │   │   ├─ emit NodeDelegatorAddedinQueue(prospectiveNodeDelegatorContract: MockNodeDelegator: [0xDDd9A038D57372934f1b9c52bd8621F5ED4268DF])
    │   │   ├─ emit NodeDelegatorAddedinQueue(prospectiveNodeDelegatorContract: MockNodeDelegator: [0x32850cAd1e9170614704fF8BA37a25e498e1B832])
    │   │   ├─ emit NodeDelegatorAddedinQueue(prospectiveNodeDelegatorContract: MockNodeDelegator: [0x1742F7d30605F18f097F356E1EeAfDB5e2FDe33a])
    │   │   └─ ← ()
    │   └─ ← ()
    ├─ [6670] TransparentUpgradeableProxy::updateMaxNodeDelegatorCount(2)
    │   ├─ [6078] LRTDepositPool::updateMaxNodeDelegatorCount(2) [delegatecall]
    │   │   ├─ [1292] TransparentUpgradeableProxy::hasRole(0x0000000000000000000000000000000000000000000000000000000000000000, admin: [0xaA10a84CE7d9AE517a52c6d5cA153b369Af99ecF]) [staticcall]
    │   │   │   ├─ [694] LRTConfig::hasRole(0x0000000000000000000000000000000000000000000000000000000000000000, admin: [0xaA10a84CE7d9AE517a52c6d5cA153b369Af99ecF]) [delegatecall]
    │   │   │   │   └─ ← true
    │   │   │   └─ ← true
    │   │   ├─ emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount: 2)
    │   │   └─ ← ()
    │   └─ ← ()
    ├─ [2633] TransparentUpgradeableProxy::getNodeDelegatorQueue() [staticcall]
    │   ├─ [2011] LRTDepositPool::getNodeDelegatorQueue() [delegatecall]
    │   │   └─ ← [0x3Ede3eCa2a72B3aeCC820E955B36f38437D01395, 0x6D9da78B6A5BEdcA287AA5d49613bA36b90c15C4, 0xDDd9A038D57372934f1b9c52bd8621F5ED4268DF, 0x32850cAd1e9170614704fF8BA37a25e498e1B832, 0x1742F7d30605F18f097F356E1EeAfDB5e2FDe33a]
    │   └─ ← [0x3Ede3eCa2a72B3aeCC820E955B36f38437D01395, 0x6D9da78B6A5BEdcA287AA5d49613bA36b90c15C4, 0xDDd9A038D57372934f1b9c52bd8621F5ED4268DF, 0x32850cAd1e9170614704fF8BA37a25e498e1B832, 0x1742F7d30605F18f097F356E1EeAfDB5e2FDe33a]
    ├─ [942] TransparentUpgradeableProxy::maxNodeDelegatorCount() [staticcall]
    │   ├─ [350] LRTDepositPool::maxNodeDelegatorCount() [delegatecall]
    │   │   └─ ← 2
    │   └─ ← 2
    └─ ← ()

Test result: ok. 1 passed; 0 failed; finished in 3.88ms
```

## Recommended mitigations
Change the limit setter function to:
```
    function updateMaxNodeDelegatorCount(uint256 maxNodeDelegatorCount_) external onlyLRTAdmin {
        require(maxNodeDelegatorCount_ >= nodeDelegatorQueue.length, "wrong limit");
        maxNodeDelegatorCount = maxNodeDelegatorCount_;
        emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount);
    }
```