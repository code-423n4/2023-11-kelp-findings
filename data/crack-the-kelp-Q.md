## Impact
Asset price is fetched from `assetPriceOracle[asset]`. If it is not set, `depositAsset` function will not work as following call [will revert](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L46) due to call to `address(0)`.
```
IPriceFetcher(assetPriceOracle[asset]).getAssetPrice(asset);
```


## Proof of Concept

```
contract LRTDepositPoolTest is BaseTest, RSETHTest {
    LRTDepositPool public lrtDepositPool;
    LRTOracle public lrtOracle;
    NodeDelegator public nodeDelegator;
    ILRTConfig public ilrtConfig;
    MockEigenStrategyManager public mockEigenStraregyManager;
    MockStrategy public mockStrategy1;
    MockStrategy public mockStrategy2;

    function setUp() public virtual override(RSETHTest, BaseTest) {
        super.setUp();
        // deploy LRTDepositPool
        ProxyAdmin proxyAdmin = new ProxyAdmin();
        LRTDepositPool contractImpl = new LRTDepositPool();
        TransparentUpgradeableProxy contractProxy = new TransparentUpgradeableProxy(
                address(contractImpl),
                address(proxyAdmin),
                ""
            );
        lrtDepositPool = LRTDepositPool(address(contractProxy));

        //deploy rsETH
        proxyAdmin = new ProxyAdmin();
        RSETH tokenImpl = new RSETH();
        TransparentUpgradeableProxy tokenProxy = new TransparentUpgradeableProxy(
                address(tokenImpl),
                address(proxyAdmin),
                ""
            );
        rseth = RSETH(address(tokenProxy));

        //deploy lrtConfig
        proxyAdmin = new ProxyAdmin();
        LRTConfig lrtConfigImpl = new LRTConfig();
        TransparentUpgradeableProxy lrtConfigProxy = new TransparentUpgradeableProxy(
                address(lrtConfigImpl),
                address(proxyAdmin),
                ""
            );

        lrtConfig = LRTConfig(address(lrtConfigProxy));

        // initialize RSETH. LRTCOnfig is already initialized in RSETHTest
        // rseth.initialize(address(admin), address(lrtConfig));
        // add rsETH to LRT config
        // lrtConfig.setRSETH(address(rseth));
        // add oracle to LRT config

        // deploy LRTOracle
        ProxyAdmin proxyAdmin1 = new ProxyAdmin();
        lrtOracle = new LRTOracle();
        TransparentUpgradeableProxy contractProxy1 = new TransparentUpgradeableProxy(
                address(lrtOracle),
                address(proxyAdmin1),
                ""
            );
        lrtOracle = LRTOracle(address(contractProxy1));

        // deploy NodeDelegator
        proxyAdmin1 = new ProxyAdmin();
        nodeDelegator = new NodeDelegator();
        contractProxy1 = new TransparentUpgradeableProxy(
            address(nodeDelegator),
            address(proxyAdmin1),
            ""
        );
        nodeDelegator = NodeDelegator(address(contractProxy1));

        lrtConfig.initialize(
            admin,
            address(stETH),
            address(rETH),
            address(cbETH),
            address(rseth)
        );
        rseth.initialize(admin, address(lrtConfig));
        lrtOracle.initialize(address(lrtConfig));
        nodeDelegator.initialize(address(lrtConfig));
        lrtDepositPool.initialize(address(lrtConfig));

        mockEigenStraregyManager = new MockEigenStrategyManager();
        mockStrategy1 = new MockStrategy(address(stETH), 0);
        mockStrategy2 = new MockStrategy(address(stETH), 0);

        vm.startPrank(admin);
        lrtConfig.setContract(
            keccak256("LRT_DEPOSIT_POOL"),
            address(lrtDepositPool)
        );
        lrtConfig.setContract(
            keccak256("EIGEN_STRATEGY_MANAGER"),
            address(mockEigenStraregyManager)
        );
        lrtConfig.grantRole(keccak256("MANAGER"), manager);
        lrtConfig.setRSETH(address(rseth));
        vm.stopPrank();

        vm.startPrank(manager);
        lrtOracle.updatePriceOracleFor(
            address(stETH),
            address(new LRTOracleMock())
        );
        lrtOracle.updatePriceOracleFor(
            address(rETH),
            address(new LRTOracleMock())
        );
        lrtOracle.updatePriceOracleFor(
            address(cbETH),
            address(new LRTOracleMock())
        );
        // lrtConfig.addNewSupportedAsset(address(randomToken), 100_000 ether);
        // lrtOracle.updatePriceOracleFor(
        //     address(randomToken),
        //     address(new LRTOracleMock())
        // );
        vm.stopPrank();

        vm.startPrank(admin);
        lrtConfig.updateAssetStrategy(address(stETH), address(mockStrategy1));

        lrtConfig.setContract(LRTConstants.LRT_ORACLE, address(lrtOracle));
        // add minter role for rseth to lrtDepositPool
        rseth.grantRole(rseth.MINTER_ROLE(), address(lrtDepositPool));

        address[] memory nodeDelegatorContracts = new address[](1);
        nodeDelegatorContracts[0] = address(nodeDelegator);
        lrtDepositPool.addNodeDelegatorContractToQueue(nodeDelegatorContracts);
        vm.stopPrank();
    }

function test_oracle_not_set() public {
        vm.startPrank(admin);
        lrtConfig.grantRole(LRTConstants.MANAGER, manager);
        vm.stopPrank();

        vm.startPrank(manager);
        lrtConfig.updateAssetDepositLimit(address(stETH), 100 ether);
        vm.stopPrank();

        vm.startPrank(alice);
        stETH.approve(address(lrtDepositPool), 5 ether);
        lrtDepositPool.depositAsset(address(stETH), 5 ether); // deposit will fail because stETH oracle not set in oracle. lrtOracle.getAssetPrice(asset) will revert and no shares can be minted reverting the transaction.
        vm.stopPrank();
    }
}
```

## Tools Used
Foundry POC

## Recommended Mitigation Steps
Consider adding a check to ensure that call reverts with proper error message.

# Use of `override` is unnecessary

As mentioned by the bot report: Starting with Solidity version [0.8.8](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding), using the override keyword when the function solely overrides an interface function, and the function doesn't exist in multiple base contracts, is unnecessary.

The report mentions that exists 13 instances of this issue but it missed the one at:

```solidity
File: src/LRTOracle.sol

20: 		        mapping(address asset => address priceOracle) public override assetPriceOracle;
```

[[20](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L20)]

# Missing function declaration in interface

The interface `INodeDelegator` is missing the declaration of the `transferBackToLRTDepositPool` function, implemented in `NodeDelegator`. Consider declaring all functions in the interface as it improves readability.
