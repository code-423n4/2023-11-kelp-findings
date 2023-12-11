---
sponsor: "Kelp DAO"
slug: "2023-11-kelp"
date: "2023-12-11"
title: "Kelp DAO | rsETH"
findings: "https://github.com/code-423n4/2023-11-kelp-findings/issues"
contest: 305
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Kelp DAO | rsETH smart contract system written in Solidity. The audit took place between November 10—November 15 2023.

## Wardens

194 Wardens contributed reports to the Kelp DAO | rsETH audit:

  1. [m\_Rassska](https://code4rena.com/@m_Rassska)
  2. [CatsSecurity](https://code4rena.com/@CatsSecurity) ([nirlin](https://code4rena.com/@nirlin) and [cats](https://code4rena.com/@cats))
  3. [adriro](https://code4rena.com/@adriro)
  4. [Bauchibred](https://code4rena.com/@Bauchibred)
  5. [PENGUN](https://code4rena.com/@PENGUN)
  6. [SBSecurity](https://code4rena.com/@SBSecurity) ([Slavcheww](https://code4rena.com/@Slavcheww) and [Blckhv](https://code4rena.com/@Blckhv))
  7. [deth](https://code4rena.com/@deth)
  8. [0xDING99YA](https://code4rena.com/@0xDING99YA)
  9. [jayfromthe13th](https://code4rena.com/@jayfromthe13th)
  10. [Aamir](https://code4rena.com/@Aamir)
  11. [HChang26](https://code4rena.com/@HChang26)
  12. [almurhasan](https://code4rena.com/@almurhasan)
  13. [d3e4](https://code4rena.com/@d3e4)
  14. [anarcheuz](https://code4rena.com/@anarcheuz)
  15. [circlelooper](https://code4rena.com/@circlelooper)
  16. [hals](https://code4rena.com/@hals)
  17. [Bauer](https://code4rena.com/@Bauer)
  18. [ck](https://code4rena.com/@ck)
  19. [Pechenite](https://code4rena.com/@Pechenite) ([Bozho](https://code4rena.com/@Bozho) and [radev\_sw](https://code4rena.com/@radev_sw))
  20. [Aymen0909](https://code4rena.com/@Aymen0909)
  21. [0xbrett8571](https://code4rena.com/@0xbrett8571)
  22. [crack-the-kelp](https://code4rena.com/@crack-the-kelp) ([Parth](https://code4rena.com/@Parth) and [wildanvin](https://code4rena.com/@wildanvin))
  23. [max10afternoon](https://code4rena.com/@max10afternoon)
  24. [hunter\_w3b](https://code4rena.com/@hunter_w3b)
  25. [lsaudit](https://code4rena.com/@lsaudit)
  26. [0xmystery](https://code4rena.com/@0xmystery)
  27. [glcanvas](https://code4rena.com/@glcanvas)
  28. [btk](https://code4rena.com/@btk)
  29. [ge6a](https://code4rena.com/@ge6a)
  30. [turvy\_fuzz](https://code4rena.com/@turvy_fuzz)
  31. [Daniel526](https://code4rena.com/@Daniel526)
  32. [Shaheen](https://code4rena.com/@Shaheen)
  33. [0xhacksmithh](https://code4rena.com/@0xhacksmithh)
  34. [0xMilenov](https://code4rena.com/@0xMilenov)
  35. [0xVolcano](https://code4rena.com/@0xVolcano)
  36. [0xepley](https://code4rena.com/@0xepley)
  37. [T1MOH](https://code4rena.com/@T1MOH)
  38. [osmanozdemir1](https://code4rena.com/@osmanozdemir1)
  39. [zhaojie](https://code4rena.com/@zhaojie)
  40. [unique](https://code4rena.com/@unique)
  41. [ast3ros](https://code4rena.com/@ast3ros)
  42. [DanielArmstrong](https://code4rena.com/@DanielArmstrong)
  43. [sakshamguruji](https://code4rena.com/@sakshamguruji)
  44. [ZanyBonzy](https://code4rena.com/@ZanyBonzy)
  45. [albahaca](https://code4rena.com/@albahaca)
  46. [foxb868](https://code4rena.com/@foxb868)
  47. [invitedtea](https://code4rena.com/@invitedtea)
  48. [chaduke](https://code4rena.com/@chaduke)
  49. [jasonxiale](https://code4rena.com/@jasonxiale)
  50. [tallo](https://code4rena.com/@tallo)
  51. [0xffchain](https://code4rena.com/@0xffchain)
  52. [deepkin](https://code4rena.com/@deepkin)
  53. [Stormy](https://code4rena.com/@Stormy)
  54. [bLnk](https://code4rena.com/@bLnk)
  55. [roleengineer](https://code4rena.com/@roleengineer)
  56. [nmirchev8](https://code4rena.com/@nmirchev8)
  57. [rouhsamad](https://code4rena.com/@rouhsamad)
  58. [GREY-HAWK-REACH](https://code4rena.com/@GREY-HAWK-REACH) ([Kose](https://code4rena.com/@Kose), [dimulski](https://code4rena.com/@dimulski), and [aslanbek](https://code4rena.com/@aslanbek),[Habib0x](https://code4rena.com/@Habib0x))
  59. [ayden](https://code4rena.com/@ayden)
  60. [rvierdiiev](https://code4rena.com/@rvierdiiev)
  61. [AlexCzm](https://code4rena.com/@AlexCzm)
  62. [adam-idarrha](https://code4rena.com/@adam-idarrha)
  63. [QiuhaoLi](https://code4rena.com/@QiuhaoLi)
  64. [Ruhum](https://code4rena.com/@Ruhum)
  65. [mahdirostami](https://code4rena.com/@mahdirostami)
  66. [xAriextz](https://code4rena.com/@xAriextz)
  67. [0x1337](https://code4rena.com/@0x1337)
  68. [Juntao](https://code4rena.com/@Juntao)
  69. [0xluckhu](https://code4rena.com/@0xluckhu)
  70. [cryptothemex](https://code4rena.com/@cryptothemex)
  71. [trachev](https://code4rena.com/@trachev)
  72. [deepplus](https://code4rena.com/@deepplus)
  73. [Weed0607](https://code4rena.com/@Weed0607)
  74. [Jiamin](https://code4rena.com/@Jiamin)
  75. [crunch](https://code4rena.com/@crunch)
  76. [Varun\_05](https://code4rena.com/@Varun_05)
  77. [7siech](https://code4rena.com/@7siech)
  78. [0xNaN](https://code4rena.com/@0xNaN)
  79. [Sathish9098](https://code4rena.com/@Sathish9098)
  80. [ABAIKUNANBAEV](https://code4rena.com/@ABAIKUNANBAEV)
  81. [0xSmartContract](https://code4rena.com/@0xSmartContract)
  82. [fouzantanveer](https://code4rena.com/@fouzantanveer)
  83. [SAAJ](https://code4rena.com/@SAAJ)
  84. [clara](https://code4rena.com/@clara)
  85. [KeyKiril](https://code4rena.com/@KeyKiril)
  86. [Myd](https://code4rena.com/@Myd)
  87. [digitizeworx](https://code4rena.com/@digitizeworx)
  88. [tala7985](https://code4rena.com/@tala7985)
  89. [Raihan](https://code4rena.com/@Raihan)
  90. [0xAnah](https://code4rena.com/@0xAnah)
  91. [JCK](https://code4rena.com/@JCK)
  92. [0xhex](https://code4rena.com/@0xhex)
  93. [0xta](https://code4rena.com/@0xta)
  94. [shamsulhaq123](https://code4rena.com/@shamsulhaq123)
  95. [Rickard](https://code4rena.com/@Rickard)
  96. [rumen](https://code4rena.com/@rumen)
  97. [spark](https://code4rena.com/@spark)
  98. [Madalad](https://code4rena.com/@Madalad)
  99. [Phantasmagoria](https://code4rena.com/@Phantasmagoria)
  100. [SandNallani](https://code4rena.com/@SandNallani)
  101. [bronze\_pickaxe](https://code4rena.com/@bronze_pickaxe)
  102. [zach](https://code4rena.com/@zach)
  103. [pep7siup](https://code4rena.com/@pep7siup)
  104. [twcctop](https://code4rena.com/@twcctop)
  105. [joaovwfreire](https://code4rena.com/@joaovwfreire)
  106. [TheSchnilch](https://code4rena.com/@TheSchnilch)
  107. [gumgumzum](https://code4rena.com/@gumgumzum)
  108. [0xrugpull\_detector](https://code4rena.com/@0xrugpull_detector)
  109. [wisdomn\_](https://code4rena.com/@wisdomn_)
  110. [ke1caM](https://code4rena.com/@ke1caM)
  111. [critical-or-high](https://code4rena.com/@critical-or-high)
  112. [ubl4nk](https://code4rena.com/@ubl4nk)
  113. [Krace](https://code4rena.com/@Krace)
  114. [peanuts](https://code4rena.com/@peanuts)
  115. [mahyar](https://code4rena.com/@mahyar)
  116. [qpzm](https://code4rena.com/@qpzm)
  117. [peter](https://code4rena.com/@peter)
  118. [SpicyMeatball](https://code4rena.com/@SpicyMeatball)
  119. [ptsanev](https://code4rena.com/@ptsanev)
  120. [Banditx0x](https://code4rena.com/@Banditx0x)
  121. [shealtielanz](https://code4rena.com/@shealtielanz)
  122. [\_thanos1](https://code4rena.com/@_thanos1)
  123. [0xAadi](https://code4rena.com/@0xAadi)
  124. [phoenixV110](https://code4rena.com/@phoenixV110)
  125. [poneta](https://code4rena.com/@poneta)
  126. [0xvj](https://code4rena.com/@0xvj)
  127. [Udsen](https://code4rena.com/@Udsen)
  128. [gesha17](https://code4rena.com/@gesha17)
  129. [Topmark](https://code4rena.com/@Topmark)
  130. [dipp](https://code4rena.com/@dipp)
  131. [adeolu](https://code4rena.com/@adeolu)
  132. [leegh](https://code4rena.com/@leegh)
  133. [cartlex\_](https://code4rena.com/@cartlex_)
  134. [RaoulSchaffranek](https://code4rena.com/@RaoulSchaffranek)
  135. [Matin](https://code4rena.com/@Matin)
  136. [cheatc0d3](https://code4rena.com/@cheatc0d3)
  137. [Tumelo\_Crypto](https://code4rena.com/@Tumelo_Crypto)
  138. [Yanchuan](https://code4rena.com/@Yanchuan)
  139. [0xHelium](https://code4rena.com/@0xHelium)
  140. [Amithuddar](https://code4rena.com/@Amithuddar)
  141. [Inspecktor](https://code4rena.com/@Inspecktor)
  142. [0xblackskull](https://code4rena.com/@0xblackskull)
  143. [0xLeveler](https://code4rena.com/@0xLeveler)
  144. [baice](https://code4rena.com/@baice)
  145. [niser93](https://code4rena.com/@niser93)
  146. [ro1sharkm](https://code4rena.com/@ro1sharkm)
  147. [hihen](https://code4rena.com/@hihen)
  148. [McToady](https://code4rena.com/@McToady)
  149. [bareli](https://code4rena.com/@bareli)
  150. [eeshenggoh](https://code4rena.com/@eeshenggoh)
  151. [seerether](https://code4rena.com/@seerether)
  152. [Soul22](https://code4rena.com/@Soul22)
  153. [AerialRaider](https://code4rena.com/@AerialRaider)
  154. [Cryptor](https://code4rena.com/@Cryptor)
  155. [boredpukar](https://code4rena.com/@boredpukar)
  156. [TeamSS](https://code4rena.com/@TeamSS) ([0xSimeon](https://code4rena.com/@0xSimeon) and [0xsagetony](https://code4rena.com/@0xsagetony))
  157. [alexfilippov314](https://code4rena.com/@alexfilippov314)
  158. [Draiakoo](https://code4rena.com/@Draiakoo)
  159. [amaechieth](https://code4rena.com/@amaechieth)
  160. [squeaky\_cactus](https://code4rena.com/@squeaky_cactus)
  161. [King\_](https://code4rena.com/@King_)
  162. [MatricksDeCoder](https://code4rena.com/@MatricksDeCoder)
  163. [Noro](https://code4rena.com/@Noro)
  164. [paritomarrr](https://code4rena.com/@paritomarrr)
  165. [soliditytaker](https://code4rena.com/@soliditytaker)
  166. [ziyou-](https://code4rena.com/@ziyou-)
  167. [codynhat](https://code4rena.com/@codynhat)
  168. [passion](https://code4rena.com/@passion)
  169. [catellatech](https://code4rena.com/@catellatech)
  170. [supersizer0x](https://code4rena.com/@supersizer0x)
  171. [stackachu](https://code4rena.com/@stackachu)
  172. [evmboi32](https://code4rena.com/@evmboi32)
  173. [taner2344](https://code4rena.com/@taner2344)
  174. [marchev](https://code4rena.com/@marchev)
  175. [Stormreckson](https://code4rena.com/@Stormreckson)
  176. [Eigenvectors](https://code4rena.com/@Eigenvectors) ([Cosine](https://code4rena.com/@Cosine) and [timo](https://code4rena.com/@timo))
  177. [ElCid](https://code4rena.com/@ElCid)
  178. [desaperh](https://code4rena.com/@desaperh)
  179. [debo](https://code4rena.com/@debo)
  180. [merlinboii](https://code4rena.com/@merlinboii)
  181. [pipidu83](https://code4rena.com/@pipidu83)
  182. [LinKenji](https://code4rena.com/@LinKenji)
  183. [zhaojohnson](https://code4rena.com/@zhaojohnson)
  184. [MaslarovK](https://code4rena.com/@MaslarovK)
  185. [Tadev](https://code4rena.com/@Tadev)

This audit was judged by [0xDjango](https://code4rena.com/@0xDjango).

Final report assembled by [liveactionllama](https://twitter.com/liveactionllama).

# Summary

The C4 analysis yielded an aggregated total of 5 unique vulnerabilities. Of these vulnerabilities, 3 received a risk rating in the category of HIGH severity and 2 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 125 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 16 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Kelp DAO | rsETH repository](https://github.com/code-423n4/2023-11-kelp), and is composed of 10 smart contracts written in the Solidity programming language and includes 498 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **Hound** from warden DadeKuma, generated the [Automated Findings report](https://gist.github.com/code423n4/12695ad1ff625d86046b7f5b78e6dda5) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# High Risk Findings (3)
## [[H-01] Possible arbitrage from Chainlink price discrepancy ](https://github.com/code-423n4/2023-11-kelp-findings/issues/584)
*Submitted by [m\_Rassska](https://github.com/code-423n4/2023-11-kelp-findings/issues/584), also found by [SBSecurity](https://github.com/code-423n4/2023-11-kelp-findings/issues/894), [Aamir](https://github.com/code-423n4/2023-11-kelp-findings/issues/865), [d3e4](https://github.com/code-423n4/2023-11-kelp-findings/issues/858), [adriro](https://github.com/code-423n4/2023-11-kelp-findings/issues/828), [CatsSecurity](https://github.com/code-423n4/2023-11-kelp-findings/issues/687), [jayfromthe13th](https://github.com/code-423n4/2023-11-kelp-findings/issues/678), [Bauchibred](https://github.com/code-423n4/2023-11-kelp-findings/issues/609), [deth](https://github.com/code-423n4/2023-11-kelp-findings/issues/516), [0xDING99YA](https://github.com/code-423n4/2023-11-kelp-findings/issues/443), [HChang26](https://github.com/code-423n4/2023-11-kelp-findings/issues/300), [almurhasan](https://github.com/code-423n4/2023-11-kelp-findings/issues/284), [PENGUN](https://github.com/code-423n4/2023-11-kelp-findings/issues/191), [anarcheuz](https://github.com/code-423n4/2023-11-kelp-findings/issues/869), and [circlelooper](https://github.com/code-423n4/2023-11-kelp-findings/issues/311)*

### Some theory needed

*   Currently KelpDAO relies on the following chainlink price feeds in order to calculate rsETH/ETH exchange rate:

|       | **Price Feed** | **Deviation** | **Heartbeat** |
| ----- | :------------: | :-----------: | :-----------: |
| **1** |    rETH/ETH    |       2%      |     86400s    |
| **2** |    cbETH/ETH   |       1%      |     86400s    |
| **3** |    stETH/ETH   |      0.5%     |     86400s    |

*   As we can see, an acceptable deviation for rETH/ETH price feed is about `[-2% 2%]`, meaning that the nodes will not update an on-chain price, in case the boundaries are not reached within the 24h period. These deviations are significant enough to open an arbitrage opportunities which will impact an overall rsETH/ETH exchange rate badly.

*   For a further analysis we have to look at the current LSD market distribution, which is represented here:

|       |   **LSD**   | **Staked ETH** | **Market Share** | **LRTDepositPool ratio** |
| ----- | :---------: | :------------: | :--------------: | ------------------------ |
| **1** |     Lido    |      8.95m     |      ~77.35%     | ~88.17%                  |
| **2** | Rocket Pool |      1.01m     |      ~8.76%      | ~9.95%                   |
| **3** |   Coinbase  |     190.549    |      ~1.65%      | ~1.88%                   |

*   Where `LRTDepositPool ratio` is an approximate ratio of deposited lsts based on the overall LSD market.

### An example of profitable arbitrage

See [original submission](https://github.com/code-423n4/2023-11-kelp-findings/issues/584) for full details.

### Final words

Basically, price feeds don't have to reach those extreme boundaries in order to profit from it. In theory presented above we were able to get +2.3% profit, which is significant in case there is a huge liquidity supplied. The combination of deviations might be absolutely random, since it operates in set of rational numbers. But it will constantly open a small \[+1%; +1.5%] arbitrage opportunities to be exploited.

### Proof on Concept

<details>

*   To reproduce the case described above, slightly change:
    *   `LRTOracleMock`:
        *   ```Solidity
            contract LRTOracleMock {
              uint256 public price;


              constructor(uint256 _price) {
                  price = _price;
              }

              function getAssetPrice(address) external view returns (uint256) {
                  return price;
              }

              function submitNewAssetPrice(uint256 _newPrice) external {
                  price = _newPrice;
              }
            }
            ```
    *   `setUp()`:
        *   ```Solidity
            contract LRTDepositPoolTest is BaseTest, RSETHTest {
            LRTDepositPool public lrtDepositPool;

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

                  // initialize RSETH. LRTCOnfig is already initialized in RSETHTest
                  rseth.initialize(address(admin), address(lrtConfig));
                  vm.startPrank(admin);
                  // add rsETH to LRT config
                  lrtConfig.setRSETH(address(rseth));
                  // add oracle to LRT config
                  lrtConfig.setContract(LRTConstants.LRT_ORACLE, address(new LRTOracle()));
                  lrtConfig.setContract(LRTConstants.LRT_DEPOSIT_POOL, address(lrtDepositPool));
                  LRTOracle(lrtConfig.getContract(LRTConstants.LRT_ORACLE)).initialize(address(lrtConfig));


                  lrtDepositPool.initialize(address(lrtConfig));
                  // add minter role for rseth to lrtDepositPool
                  rseth.grantRole(rseth.MINTER_ROLE(), address(lrtDepositPool));

              }
            }
            ```
    *   `test_DepositAsset()`:
        *   ```Solidity
                function test_DepositAsset() external {
                  address rETHPriceOracle = address(new LRTOracleMock(1.09149e18));
                  address stETHPriceOracle = address(new LRTOracleMock(0.99891e18));
                  address cbETHPriceOracle = address(new LRTOracleMock(1.05407e18));
                  LRTOracle(lrtConfig.getContract(LRTConstants.LRT_ORACLE)).updatePriceOracleFor(address(rETH), rETHPriceOracle);
                  LRTOracle(lrtConfig.getContract(LRTConstants.LRT_ORACLE)).updatePriceOracleFor(address(stETH), stETHPriceOracle);
                  LRTOracle(lrtConfig.getContract(LRTConstants.LRT_ORACLE)).updatePriceOracleFor(address(cbETH), cbETHPriceOracle);

                  console.log("rETH/ETH exchange rate after", LRTOracleMock(rETHPriceOracle).getAssetPrice(address(0)));
                  console.log("stETH/ETH exchange rate after", LRTOracleMock(stETHPriceOracle).getAssetPrice(address(0)));
                  console.log("cbETH/ETH exchange rate after", LRTOracleMock(cbETHPriceOracle).getAssetPrice(address(0)));

                  vm.startPrank(alice); // alice provides huge amount of liquidity to the pool

                  rETH.approve(address(lrtDepositPool), 9950 ether);
                  lrtDepositPool.depositAsset(rETHAddress, 9950 ether);

                  stETH.approve(address(lrtDepositPool), 88170 ether);
                  lrtDepositPool.depositAsset(address(stETH), 88170 ether);

                  cbETH.approve(address(lrtDepositPool), 1880 ether);
                  lrtDepositPool.depositAsset(address(cbETH), 1880 ether);

                  vm.stopPrank();


                  vm.startPrank(carol); // carol deposits, when the price feeds return answer pretty close to a spot price

                  uint256 carolBalanceBefore = rseth.balanceOf(address(carol));

                  rETH.approve(address(lrtDepositPool), 100 ether);
                  lrtDepositPool.depositAsset(address(rETH), 100 ether);

                  uint256 carolBalanceAfter = rseth.balanceOf(address(carol));

                  vm.stopPrank();

                  uint256 rETHNewPrice = uint256(LRTOracleMock(rETHPriceOracle).getAssetPrice(address(0))) * 102 / 100; // +2%
                  uint256 stETHNewPrice = uint256(LRTOracleMock(stETHPriceOracle).getAssetPrice(address(0))) * 995 / 1000; // -0.5%
                  uint256 cbETHNewPrice = uint256(LRTOracleMock(cbETHPriceOracle).getAssetPrice(address(0))) * 99 / 100; // -1%

                  LRTOracleMock(rETHPriceOracle).submitNewAssetPrice(rETHNewPrice);
                  LRTOracleMock(stETHPriceOracle).submitNewAssetPrice(stETHNewPrice);
                  LRTOracleMock(cbETHPriceOracle).submitNewAssetPrice(cbETHNewPrice);

                  console.log("rETH/ETH exchange rate after", LRTOracleMock(rETHPriceOracle).getAssetPrice(address(0)));
                  console.log("stETH/ETH exchange rate after", LRTOracleMock(stETHPriceOracle).getAssetPrice(address(0)));
                  console.log("cbETH/ETH exchange rate after", LRTOracleMock(cbETHPriceOracle).getAssetPrice(address(0)));

                  vm.startPrank(bob);

                  // bob balance of rsETH before deposit
                  uint256 bobBalanceBefore = rseth.balanceOf(address(bob));

                  rETH.approve(address(lrtDepositPool), 100 ether);
                  lrtDepositPool.depositAsset(address(rETH), 100 ether);

                  uint256 bobBalanceAfter = rseth.balanceOf(address(bob));
                  vm.stopPrank();

                  assertEq(bobBalanceBefore, carolBalanceBefore, "the balances are not the same");
                  assertGt(bobBalanceAfter, carolBalanceAfter * 102 / 100, "some random shit happened");
                  assertLt(bobBalanceAfter, carolBalanceAfter * 103 / 100, "some random shittttt happened");

                }
            ```

</details>

### Recommended Mitigation Steps

**Short term:**

*   N/A

**Long term:**

*   I was thinking about utilizing multiple price oracles, which could potentially close any profitable opportunities, but the gas overhead and overall complexity grows rapidly. Unfortunately, I don't have anything robust to offer by now, but open to discuss about it.

### Assessed type

Math

**[gus (Kelp) disputed and commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/584#issuecomment-1825309294):**
 > We appreciate the explanation at length you have here. We see the arbitrage as a feature rather than a bug in the system. The past 2 year data on the price discrepancy assures us that this is a minor vector that will not impact deposits or withdraws significantly. Moreover, the fact that minters earn nominally more and withdrawals earn nominally less balances the system. We also want to call out that price arbitrage is not a unique problem to our design. It is virtually present across every staking protocol and we encourage healthy arbitrage opportunity here.

**[0xDjango (judge) commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/584#issuecomment-1847574552):**
 > This is valid. Depositors will be able to deposit the minimally-priced asset and steal value from the deposit pool. The deviation % and heartbeat are too large and this will open up arbitrage opportunities to the detriment of Kelp's users. When Kelp implements the withdrawal mechanism, we will have a better understanding of the profitability of such an attack. For example, if Kelp implements withdrawals without a staking delay, given the large amount of on-chain liquidity of the underlying assets, the pool may be able to be exploited for large amounts given even a 1% price discrepancy between the different LSTs.

*Note: see [original submission](https://github.com/code-423n4/2023-11-kelp-findings/issues/584) for full discussion.*



***

## [[H-02] Protocol mints less rsETH on deposit than intended](https://github.com/code-423n4/2023-11-kelp-findings/issues/62)
*Submitted by [T1MOH](https://github.com/code-423n4/2023-11-kelp-findings/issues/62), also found by [0xepley](https://github.com/code-423n4/2023-11-kelp-findings/issues/863), [SBSecurity](https://github.com/code-423n4/2023-11-kelp-findings/issues/860), [cryptothemex](https://github.com/code-423n4/2023-11-kelp-findings/issues/853), [adriro](https://github.com/code-423n4/2023-11-kelp-findings/issues/817), [AlexCzm](https://github.com/code-423n4/2023-11-kelp-findings/issues/777), [trachev](https://github.com/code-423n4/2023-11-kelp-findings/issues/765), [adam-idarrha](https://github.com/code-423n4/2023-11-kelp-findings/issues/761), [Aymen0909](https://github.com/code-423n4/2023-11-kelp-findings/issues/694), [deepplus](https://github.com/code-423n4/2023-11-kelp-findings/issues/672), [xAriextz](https://github.com/code-423n4/2023-11-kelp-findings/issues/636), [ast3ros](https://github.com/code-423n4/2023-11-kelp-findings/issues/594), [Weed0607](https://github.com/code-423n4/2023-11-kelp-findings/issues/549), [DanielArmstrong](https://github.com/code-423n4/2023-11-kelp-findings/issues/542), [rouhsamad](https://github.com/code-423n4/2023-11-kelp-findings/issues/540), [osmanozdemir1](https://github.com/code-423n4/2023-11-kelp-findings/issues/524), [GREY-HAWK-REACH](https://github.com/code-423n4/2023-11-kelp-findings/issues/503), [0x1337](https://github.com/code-423n4/2023-11-kelp-findings/issues/490), [zhaojie](https://github.com/code-423n4/2023-11-kelp-findings/issues/455), [Jiamin](https://github.com/code-423n4/2023-11-kelp-findings/issues/449), [crunch](https://github.com/code-423n4/2023-11-kelp-findings/issues/427), [Varun\_05](https://github.com/code-423n4/2023-11-kelp-findings/issues/402), [7siech](https://github.com/code-423n4/2023-11-kelp-findings/issues/379), [QiuhaoLi](https://github.com/code-423n4/2023-11-kelp-findings/issues/344), [circlelooper](https://github.com/code-423n4/2023-11-kelp-findings/issues/312), [HChang26](https://github.com/code-423n4/2023-11-kelp-findings/issues/291), [Juntao](https://github.com/code-423n4/2023-11-kelp-findings/issues/275), [ayden](https://github.com/code-423n4/2023-11-kelp-findings/issues/274), [Aamir](https://github.com/code-423n4/2023-11-kelp-findings/issues/256), [rvierdiiev](https://github.com/code-423n4/2023-11-kelp-findings/issues/200), [max10afternoon](https://github.com/code-423n4/2023-11-kelp-findings/issues/181), [crack-the-kelp](https://github.com/code-423n4/2023-11-kelp-findings/issues/175), [Ruhum](https://github.com/code-423n4/2023-11-kelp-findings/issues/173), [0xluckhu](https://github.com/code-423n4/2023-11-kelp-findings/issues/164), [0xNaN](https://github.com/code-423n4/2023-11-kelp-findings/issues/105), [mahdirostami](https://github.com/code-423n4/2023-11-kelp-findings/issues/98), and [0xmystery](https://github.com/code-423n4/2023-11-kelp-findings/issues/95)*

Price of rsETH is calculated as `totalLockedETH / rsETHSupply`. rsETH price is used to calculate rsETH amount to mint when user deposits. Formulas are following:

*   `rsethAmountToMint = amount * assetPrice / rsEthPrice`
*   `rsEthPrice = totalEthLocked / rsETHSupply`

Problem is that it transfers deposit amount before calculation of `rsethAmountToMint`. It increases `totalEthLocked`. As a result rsethAmountToMint is less than intended because rsEthPrice is higher.

For example:

1.  Suppose `totalEthLocked` = 10e18, assetPrice = 1e18, rsETHSupply = 10e18
2.  User deposits 30e18. He expects to receive 30e18 rsETH
3.  However actual received amount will be `30e18 * 1e18 / ((30e18 * 1e18 + 10e18 * 1e18) / 10e18) = 7.5e18`

### Proof of Concept

Here you can see that it firstly transfers asset to `address(this)`, then calculates amount to mint:

```solidity
    function depositAsset(
        address asset,
        uint256 depositAmount
    )
        external
        whenNotPaused
        nonReentrant
        onlySupportedAsset(asset)
    {
        ...

        if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) {
            revert TokenTransferFailed();
        }

        // interactions
        uint256 rsethAmountMinted = _mintRsETH(asset, depositAmount);

        emit AssetDeposit(asset, depositAmount, rsethAmountMinted);
    }
```

There is long chain of calls:

```solidity
_mintRsETH()
    getRsETHAmountToMint()
        LRTOracle().getRSETHPrice()
            getTotalAssetDeposits()
                getTotalAssetDeposits()
```

Finally `getTotalAssetDeposits()` uses current `balanceOf()`, which was increased before by transferring deposit amount:

```solidity
    function getAssetDistributionData(address asset)
        public
        view
        override
        onlySupportedAsset(asset)
        returns (uint256 assetLyingInDepositPool, uint256 assetLyingInNDCs, uint256 assetStakedInEigenLayer)
    {
        // Question: is here the right place to have this? Could it be in LRTConfig?
@>      assetLyingInDepositPool = IERC20(asset).balanceOf(address(this));

        uint256 ndcsCount = nodeDelegatorQueue.length;
        for (uint256 i; i < ndcsCount;) {
            assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
            unchecked {
                ++i;
            }
        }
    }
```

### Recommended Mitigation Steps

Transfer tokens in the end:

```diff
    function depositAsset(
        address asset,
        uint256 depositAmount
    )
        external
        whenNotPaused
        nonReentrant
        onlySupportedAsset(asset)
    {
        ...

+       uint256 rsethAmountMinted = _mintRsETH(asset, depositAmount);

        if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) {
            revert TokenTransferFailed();
        }

-       // interactions
-       uint256 rsethAmountMinted = _mintRsETH(asset, depositAmount);
-
        emit AssetDeposit(asset, depositAmount, rsethAmountMinted);
    }
```

### Assessed type

Oracle

**[RaymondFam (lookout) commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/62#issuecomment-1813278876):**
 > This vulnerability is worse than a donation attack.

**[gus (Kelp) confirmed and commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/62#issuecomment-1850480282):**
 >This is a legitimate issue and has been fixed in commit 3b4e36c740013b32b78e93b00438b25f848e5f76 to separately have rsETH price calculators and read value from state variables, which also helped reduce gas cost to a great extent as well. We thank the warden for alerting us to this issue.

**[0xDjango (judge) commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/62#issuecomment-1838891636):**
 > This is a High severity issue. The miscalculation causes direct fund loss to users.

***

## [[H-03] The price of rsETH could be manipulated by the first staker](https://github.com/code-423n4/2023-11-kelp-findings/issues/42)
*Submitted by [Krace](https://github.com/code-423n4/2023-11-kelp-findings/issues/42), also found by [SBSecurity](https://github.com/code-423n4/2023-11-kelp-findings/issues/855), [spark](https://github.com/code-423n4/2023-11-kelp-findings/issues/842), [adriro](https://github.com/code-423n4/2023-11-kelp-findings/issues/819), [Madalad](https://github.com/code-423n4/2023-11-kelp-findings/issues/811), [peanuts](https://github.com/code-423n4/2023-11-kelp-findings/issues/751), [adam-idarrha](https://github.com/code-423n4/2023-11-kelp-findings/issues/733), [Phantasmagoria](https://github.com/code-423n4/2023-11-kelp-findings/issues/727), [Aymen0909](https://github.com/code-423n4/2023-11-kelp-findings/issues/698), [CatsSecurity](https://github.com/code-423n4/2023-11-kelp-findings/issues/676), [SandNallani](https://github.com/code-423n4/2023-11-kelp-findings/issues/666), [chaduke](https://github.com/code-423n4/2023-11-kelp-findings/issues/658), [AlexCzm](https://github.com/code-423n4/2023-11-kelp-findings/issues/631), [deth](https://github.com/code-423n4/2023-11-kelp-findings/issues/621), [bronze\_pickaxe](https://github.com/code-423n4/2023-11-kelp-findings/issues/601), [osmanozdemir1](https://github.com/code-423n4/2023-11-kelp-findings/issues/598), [ast3ros](https://github.com/code-423n4/2023-11-kelp-findings/issues/595), [zach](https://github.com/code-423n4/2023-11-kelp-findings/issues/588), [m\_Rassska](https://github.com/code-423n4/2023-11-kelp-findings/issues/582), [jasonxiale](https://github.com/code-423n4/2023-11-kelp-findings/issues/570), [mahyar](https://github.com/code-423n4/2023-11-kelp-findings/issues/559), [pep7siup](https://github.com/code-423n4/2023-11-kelp-findings/issues/557), [twcctop](https://github.com/code-423n4/2023-11-kelp-findings/issues/547), [ck](https://github.com/code-423n4/2023-11-kelp-findings/issues/532), [joaovwfreire](https://github.com/code-423n4/2023-11-kelp-findings/issues/528), [GREY-HAWK-REACH](https://github.com/code-423n4/2023-11-kelp-findings/issues/511), [0xDING99YA](https://github.com/code-423n4/2023-11-kelp-findings/issues/451), [zhaojie](https://github.com/code-423n4/2023-11-kelp-findings/issues/441), [qpzm](https://github.com/code-423n4/2023-11-kelp-findings/issues/430), [TheSchnilch](https://github.com/code-423n4/2023-11-kelp-findings/issues/377), [gumgumzum](https://github.com/code-423n4/2023-11-kelp-findings/issues/353), [QiuhaoLi](https://github.com/code-423n4/2023-11-kelp-findings/issues/330), [0xrugpull\_detector](https://github.com/code-423n4/2023-11-kelp-findings/issues/305), [Aamir](https://github.com/code-423n4/2023-11-kelp-findings/issues/297), [almurhasan](https://github.com/code-423n4/2023-11-kelp-findings/issues/286), [ayden](https://github.com/code-423n4/2023-11-kelp-findings/issues/281), [wisdomn\_](https://github.com/code-423n4/2023-11-kelp-findings/issues/264), [peter](https://github.com/code-423n4/2023-11-kelp-findings/issues/257), [max10afternoon](https://github.com/code-423n4/2023-11-kelp-findings/issues/241), [rouhsamad](https://github.com/code-423n4/2023-11-kelp-findings/issues/219), [SpicyMeatball](https://github.com/code-423n4/2023-11-kelp-findings/issues/218), [rvierdiiev](https://github.com/code-423n4/2023-11-kelp-findings/issues/204), [crack-the-kelp](https://github.com/code-423n4/2023-11-kelp-findings/issues/203), [Ruhum](https://github.com/code-423n4/2023-11-kelp-findings/issues/174), [ke1caM](https://github.com/code-423n4/2023-11-kelp-findings/issues/172), [ptsanev](https://github.com/code-423n4/2023-11-kelp-findings/issues/118), [mahdirostami](https://github.com/code-423n4/2023-11-kelp-findings/issues/99), [Banditx0x](https://github.com/code-423n4/2023-11-kelp-findings/issues/85), [critical-or-high](https://github.com/code-423n4/2023-11-kelp-findings/issues/78), [T1MOH](https://github.com/code-423n4/2023-11-kelp-findings/issues/60), [ubl4nk](https://github.com/code-423n4/2023-11-kelp-findings/issues/53), [Bauer](https://github.com/code-423n4/2023-11-kelp-findings/issues/45), and [btk](https://github.com/code-423n4/2023-11-kelp-findings/issues/788)*

<https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTDepositPool.sol#L95-L110><br>
<https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTOracle.sol#L52-L79>

The first staker can potentially manipulate the price of rsETH through a donation attack, causing subsequent stakers to receive no rsETH after depositing. The first staker can exploit this method to siphon funds from other users.

### Proof of Concept

The mining amount of rsETH is calculated in function `getRsETHAmountToMint` which directly utilizes the total value of the asset divided by the price of a single rsETH.

```solidity
    function getRsETHAmountToMint(
        address asset,
        uint256 amount
    )
        public
        view
        override
        returns (uint256 rsethAmountToMint)
    {
        // setup oracle contract
        address lrtOracleAddress = lrtConfig.getContract(LRTConstants.LRT_ORACLE);
        ILRTOracle lrtOracle = ILRTOracle(lrtOracleAddress);

        // calculate rseth amount to mint based on asset amount and asset exchange rate
        rsethAmountToMint = (amount * lrtOracle.getAssetPrice(asset)) / lrtOracle.getRSETHPrice();
    }
```

Subsequently, the price of rsETH is related to its totalSupply and the total value of deposited assets.

```solidity
    function getRSETHPrice() external view returns (uint256 rsETHPrice) {
        address rsETHTokenAddress = lrtConfig.rsETH();
        uint256 rsEthSupply = IRSETH(rsETHTokenAddress).totalSupply();

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

            uint256 totalAssetAmt = ILRTDepositPool(lrtDepositPoolAddr).getTotalAssetDeposits(asset);
            totalETHInPool += totalAssetAmt * assetER;

            unchecked {
                ++asset_idx;
            }
        }
//@audit the price of rsETH is calculated based on the asset and totalSupply
        return totalETHInPool / rsEthSupply;
    }
```

The total value of deposited assets comprises three parts: the assets in `LRTDepositPool`, the assets in `NodeDelagator`, and the assets in the eigenlayer. Anyone can directly contribute asset tokens to `LRTDepositPool` or `NodeDelegator` to augment the total value of deposited assets.

```solidity
    function getAssetDistributionData(address asset)
        public
        view
        override
        onlySupportedAsset(asset)
        returns (uint256 assetLyingInDepositPool, uint256 assetLyingInNDCs, uint256 assetStakedInEigenLayer)
    {
        // Question: is here the right place to have this? Could it be in LRTConfig?
        assetLyingInDepositPool = IERC20(asset).balanceOf(address(this));

        uint256 ndcsCount = nodeDelegatorQueue.length;
        for (uint256 i; i < ndcsCount;) {
            assetLyingInNDCs += IERC20(asset).balanceOf(nodeDelegatorQueue[i]);
            assetStakedInEigenLayer += INodeDelegator(nodeDelegatorQueue[i]).getAssetBalance(asset);
            unchecked {
                ++i;
            }
        }
    }
```

### Case

Therefore, the price of rsETH is susceptible to manipulation by the first staker, considering the following scenario:

- Alice is the first staker and she deposits 1 USDC (the price of USDC is set to \$1), she will get 1 wei rsETH, and the totalSupply of rsETH is 1 wei. 

Here is the test, add it to `test/LRTDepositPoolTest.t.sol` and run with `forge test --match-test test_ControlPrice -vv`.

```diff
diff --git a/test/LRTDepositPoolTest.t.sol b/test/LRTDepositPoolTest.t.sol
index 40abc93..63349c2 100644
--- a/test/LRTDepositPoolTest.t.sol
+++ b/test/LRTDepositPoolTest.t.sol
@@ -9,10 +9,11 @@ import { ILRTDepositPool } from "src/interfaces/ILRTDepositPool.sol";

 import { TransparentUpgradeableProxy } from "@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol";
 import { ProxyAdmin } from "@openzeppelin/contracts/proxy/transparent/ProxyAdmin.sol";
+import "forge-std/console.sol";

 contract LRTOracleMock {
     function getAssetPrice(address) external pure returns (uint256) {
-        return 1e18;
+        return 1;
     }

     function getRSETHPrice() external pure returns (uint256) {
@@ -109,6 +110,23 @@ contract LRTDepositPoolDepositAsset is LRTDepositPoolTest {
         lrtDepositPool.depositAsset(rETHAddress, 2 ether);
     }

+    function test_ControlPrice() external {
+        vm.startPrank(alice);
+
+        // alice balance of rsETH before deposit
+        uint256 aliceBalanceBefore = rseth.balanceOf(address(alice));
+
+        rETH.approve(address(lrtDepositPool), 1 ether);
+        lrtDepositPool.depositAsset(rETHAddress, 1 ether);
+
+        // alice balance of rsETH after deposit
+        uint256 aliceBalanceAfter = rseth.balanceOf(address(alice));
+        vm.stopPrank();
+
+        console.log(" rsETH of Alice: ", aliceBalanceAfter - aliceBalanceBefore);
+
+    }
+
     function test_DepositAsset() external {
         vm.startPrank(alice);
```

*   Alice donates 10000 USDC to the `LRTDepositPool` to inflate the price of rsETH. Now the price of rsETH is: (10000 + 1)ether / 1 wei = 10001 ether

*   Any subsequent staker who deposits assets worth less than 10001 USDC will not receive any rsETH, and they won't be able to withdraw the deposited assets either. Alice can directly siphon off these funds.

For example, if Bob deposit 10000 USDC, then the `rsethAmountToMint` is `(10000 ether * 1) / (10001)ether = 0`

    rsethAmountToMint = (amount * lrtOracle.getAssetPrice(asset)) / lrtOracle.getRSETHPrice();

Moreover, there is no check on the actual amount of rsETH received by the user, and the execution continues even if this amount is zero.

        function _mintRsETH(address _asset, uint256 _amount) private returns (uint256 rsethAmountToMint) {
            (rsethAmountToMint) = getRsETHAmountToMint(_asset, _amount);

            address rsethToken = lrtConfig.rsETH();
            // mint rseth for user
            //@audit sender could receive 0 token
            IRSETH(rsethToken).mint(msg.sender, rsethAmountToMint);
        }

### Recommended Mitigation Steps

It is recommended to pre-mint some rsETH tokens to prevent price manipulation or ensure that the `rsethAmountToMint` is greater than zero.

**[gus (Kelp) disagreed with severity and commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/42#issuecomment-1825350085):**
 > We agree this is an issue. We also agree that it should be of a MEDIUM severity as it is an edge case that happens on the first protocol interaction.

**[manoj9april (Kelp) confirmed](https://github.com/code-423n4/2023-11-kelp-findings/issues/42#issuecomment-1825451322)**

**[0xDjango (judge) commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/42#issuecomment-1836453373):**
 > Judging as HIGH. While it is an edge case, the potential loss of funds is present. Vault donation attacks have been judged as high in the majority of C4 audits where no safeguards are implemented.

**[manoj9april (Kelp) commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/42#issuecomment-1837395661):**
 > Initial minting is a way of mitigating this issue. And this mitigation could be done after deployment. Hence no safeguard were added in contract. Hence request to decrease to medium.

**[0xDjango (judge) commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/42#issuecomment-1838882957):**
 > > *Initial minting is a way of mitigating this issue. And this mitigation could be done after deployment. Hence no safeguard were added in contract. Hence request to decrease to medium.*
> 
> Based on the implementation, this issue will remain HIGH. Funds are at risk until Kelp takes subsequent action to mitigate.

**[gus (Kelp) confirmed and commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/62#issuecomment-1850480282):**
 >We disagree with the severity of this issue. Every protocol has to setup the contracts first before publicizing that contracts are ready for public usage. We take measures to ensure the exchange rate is closer to 1 before users interact with contracts.

***

 
# Medium Risk Findings (2)
## [[M-01] Update in strategy will cause wrong issuance of shares](https://github.com/code-423n4/2023-11-kelp-findings/issues/197)
*Submitted by [crack-the-kelp](https://github.com/code-423n4/2023-11-kelp-findings/issues/197), also found by [crack-the-kelp](https://github.com/code-423n4/2023-11-kelp-findings/issues/198), [osmanozdemir1](https://github.com/code-423n4/2023-11-kelp-findings/issues/893), [T1MOH](https://github.com/code-423n4/2023-11-kelp-findings/issues/833), [tallo](https://github.com/code-423n4/2023-11-kelp-findings/issues/821), [0xffchain](https://github.com/code-423n4/2023-11-kelp-findings/issues/818), [Stormy](https://github.com/code-423n4/2023-11-kelp-findings/issues/797), [jayfromthe13th](https://github.com/code-423n4/2023-11-kelp-findings/issues/775), [Aymen0909](https://github.com/code-423n4/2023-11-kelp-findings/issues/688), [bLnk](https://github.com/code-423n4/2023-11-kelp-findings/issues/649), [Pechenite](https://github.com/code-423n4/2023-11-kelp-findings/issues/644), [chaduke](https://github.com/code-423n4/2023-11-kelp-findings/issues/638), [roleengineer](https://github.com/code-423n4/2023-11-kelp-findings/issues/620), [ast3ros](https://github.com/code-423n4/2023-11-kelp-findings/issues/596), [jasonxiale](https://github.com/code-423n4/2023-11-kelp-findings/issues/564), [deth](https://github.com/code-423n4/2023-11-kelp-findings/issues/518), [ZanyBonzy](https://github.com/code-423n4/2023-11-kelp-findings/issues/513), [zhaojie](https://github.com/code-423n4/2023-11-kelp-findings/issues/487), [0xDING99YA](https://github.com/code-423n4/2023-11-kelp-findings/issues/454), [Bauchibred](https://github.com/code-423n4/2023-11-kelp-findings/issues/426), [lsaudit](https://github.com/code-423n4/2023-11-kelp-findings/issues/419), [deepkin](https://github.com/code-423n4/2023-11-kelp-findings/issues/398), [DanielArmstrong](https://github.com/code-423n4/2023-11-kelp-findings/issues/339), and [nmirchev8](https://github.com/code-423n4/2023-11-kelp-findings/issues/334)*

<https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L70><br>
<https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L49><br>
<https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L84><br>
<https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L122-L123>

If there is update in strategy, `getTotalAssetDeposits(asset)` will reduce for asset. Because it checks for the [assets deposited in NodeDelegator](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L84). NodeDelegator.getAssetBalance() will check for asset balance for the strategy of the user. If strategy is updated, then the balance of old strategy won't be taken into account which will reduce the totalDeposits. This will reduce the rsETH share price and cause more rsETH shares to be minted for the depositors. Thus, depositors can immediately deposit and their shares are worth more than their deposit.

### Proof of Concept

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

    function test_update_strategy_cause_wrong_minting_of_shares() public {
            //NOTE: comment line 207-211 in setup for this
            vm.startPrank(admin);
            lrtConfig.grantRole(LRTConstants.MANAGER, manager);
            vm.stopPrank();
            vm.startPrank(alice);
            stETH.approve(address(lrtDepositPool), 10 ether);
            lrtDepositPool.depositAsset(address(stETH), 10 ether); //NOTE: 10000000000000000000 rsETH amount minted
            vm.stopPrank();
            vm.startPrank(manager);
            lrtDepositPool.transferAssetToNodeDelegator(
                0,
                address(stETH),
                10 ether
            );
            vm.stopPrank();
            vm.startPrank(admin);
            lrtConfig.updateAssetStrategy(address(stETH), address(mockStrategy2));
            lrtConfig.updateAssetStrategy(address(rETH), address(mockStrategy2));
            lrtConfig.updateAssetStrategy(address(cbETH), address(mockStrategy2));

            vm.stopPrank();
            vm.startPrank(alice);
            stETH.approve(address(lrtDepositPool), 10 ether);
            lrtDepositPool.depositAsset(address(stETH), 10 ether); //NOTE:   2500000000000000000 rsETH amount minted even after fixing deposit bug
            vm.stopPrank();
        }
    }

### Tools Used

Foundry

### Recommended Mitigation Steps

While updating the strategy, ensure that balance deposited in previous strategy for same asset is accounted while minting the shares.

### Assessed type

ERC4626

**[gus (Kelp) disputed and commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/197#issuecomment-1825335776):**
 > The Eigenlayer has a 1-1 mapping for asset and its strategy.  We don't believe this an issue with KelpDAO or its code but with Eigenlayer if it upgrades its protocol to have multiple strategies for an asset.  Then if this occurs, KelpDAO will also have to upgrade its contracts to ensure the compatibility to Eigenlayer.<br>
> Also, if Eigenlayer changes strategies, they are responsible to migrate the internal accounting to the new address. 

**[manoj9april (Kelp) commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/197#issuecomment-1833168052):**
 > Currently Eigenlayer's implementation allows them to have multiple startegies for any asset.<br>
> -> But we can continue to deposit into one strategy for a asset and it won't be a problem for us. That is assuming 1:1 mapping won't harm us.
> 
> Above assumptions will only create problem in case, Eigenlayer wants to spread the current funds into multiple strategies for any asset in future upgrades. And in this case, they will provide proper heads up ahead of time and we can upgrade along with them accordingly.
> 
> In case they change their strategy for any asset, we can point to new strategy address and it should work fine. And in this case Eigenlayer should should take care of all migration of funds from old to new strategy.
> 
> But as currently nothing has been decided at Eigenlayer end. I believe we are good with our current assumption of 1:1. It would be overkill to assume anything else as of now.
> 
> Please check the attached conversation with Eigenlayer team.<br>
> Discord: https://discord.com/channels/1089434273720832071/1096331191243788379

**[0xDjango (judge) invalidated and commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/197#issuecomment-1836498666):**
 > Kelp has provided adequate reasoning behind the current design. I agree that it is unlikely that a breaking change from the Eigenlayer protocol would not be properly communicated ahead of time. Designing around this potentiality would impose added risk in the current design.

**[radev\_sw (warden) commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/197#issuecomment-1837271970):**
 > Hey, @0xDjango.
> 
> I disagree with the reasons provided above.
> 
> > *We don't believe this an issue with KelpDAO or its code but with Eigenlayer if it upgrades its protocol to have multiple strategies for an asset.*
> 
> The problem lies specifically in how Kelp DAO upgrades the strategy for assets. Nothing related to the functionality of EigenLayer Strategies.
> 
> Let's view at the code of startegy upgrading
> ```js
>     function updateAssetStrategy(
>         address asset,
>         address strategy
>     )
>         external
>         onlyRole(DEFAULT_ADMIN_ROLE)
>         onlySupportedAsset(asset)
>     {
>         UtilLib.checkNonZeroAddress(strategy);
>         if (assetStrategy[asset] == strategy) {
>             revert ValueAlreadyInUse();
>         }
>         assetStrategy[asset] = strategy;
>     }
> ```
> 
> As we can see, when an EigenLayer Strategy for an asset is changed, there is currently no mechanism in place to migrate user deposits from the old strategy to the new one. It is precisely in this scenario that the vulnerability arises.
> 
> If a strategy update occurs, the function `getTotalAssetDeposits(asset)` will show a reduced value for that asset. This reduction happens because the function references assets deposited in NodeDelegator. Specifically, `NodeDelegator.sol#getAssetBalance()` assesses the asset balance based on the user's current strategy. When a strategy update takes place, the balance associated with the previous strategy is no longer considered, leading to a decrease in `totalDeposits`. As a result, the share price of `rsETH` decreases, prompting the system to mint more `rsETH` shares for depositors. Consequently, depositors can make immediate deposits, gaining shares that are more valuable than the actual amount they deposited.
> 
> This vulnerability describes **significant flaws** in the KelpDAO protocol flow. Nothing related to the functionality of EigenLayer Strategies.
> 
> > *But as currently nothing has been decided at Eigenlayer end. I believe we are good with our current assumption of 1:1. It would be a overkill to assume anything else as of now.*
> 
> The recommendations in this report do not make any assumptions about the implementation of 1:1 mapping. Instead, they specifically suggest updating the `updateAssetStrategy()` function. In a way that update would enable the transfer of user deposits from the old strategy to the new strategy once the new strategy is set. (as in my issue [\#644](https://github.com/code-423n4/2023-11-kelp-findings/issues/644) writes)
> 
> I believe the issue has been misjudged and should be mitigated to High Severity Issue.

**[manoj9april (Kelp) commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/197#issuecomment-1837404596):**
 > @radev\_sw - KelpDao will only update the strategy address when EigenLayer updates strategy for any asset (which EigenLayer currently has no plans as such).
> 
> Even if above scenario takes place. EigenLayer should provide proper communications ahead of time. And we can go through follow below steps:
> - Eigenlayer communicates strategy update
> - KelpDao pauses its protocol for a mutually decided duration
> - EigenLayer migrates its state from old contracts to new contracts
> - KelpDao points to new strategy contract
> - KelpDao resumes
> 
> If migration of contract state is not possible at EigenLayer and they (Eigenlayer) decomission their old strategy, only in this scenario<br>
> KelpDao will have to withdraw its funds from old strategy and redelgate it to new strategy. And these steps will be done in complete visibility and transparency with proper timelocks.
> But sees no harm or loss to funds or miscalculation.

**[0xDjango (judge) increased severity to Medium and commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/197#issuecomment-1839037847):**
 > This will be upgraded to MEDIUM. 
> 
> The warden accurately explains how **Kelp's** internal accounting will be broken if they decide to use the `updateAssetStrategy()` function. If Kelp changes the strategy address, rsETH share prices will decrease in a way that should not occur.
> 
> While Kelp describes the list of actions that they would take in the case they would need to change the strategy address, the implementation does not protect against the possibility of the broken accounting.



***

## [[M-02] Lack of slippage control on `LRTDepositPool.depositAsset`](https://github.com/code-423n4/2023-11-kelp-findings/issues/148)
*Submitted by [max10afternoon](https://github.com/code-423n4/2023-11-kelp-findings/issues/148), also found by [0xhacksmithh](https://github.com/code-423n4/2023-11-kelp-findings/issues/884), [ge6a](https://github.com/code-423n4/2023-11-kelp-findings/issues/862), [btk](https://github.com/code-423n4/2023-11-kelp-findings/issues/847), [anarcheuz](https://github.com/code-423n4/2023-11-kelp-findings/issues/846), [adriro](https://github.com/code-423n4/2023-11-kelp-findings/issues/825), [Aymen0909](https://github.com/code-423n4/2023-11-kelp-findings/issues/681), [Pechenite](https://github.com/code-423n4/2023-11-kelp-findings/issues/647), [0xMilenov](https://github.com/code-423n4/2023-11-kelp-findings/issues/611), [turvy\_fuzz](https://github.com/code-423n4/2023-11-kelp-findings/issues/550), ck ([1](https://github.com/code-423n4/2023-11-kelp-findings/issues/522), [2](https://github.com/code-423n4/2023-11-kelp-findings/issues/510)), hals ([1](https://github.com/code-423n4/2023-11-kelp-findings/issues/412), [2](https://github.com/code-423n4/2023-11-kelp-findings/issues/409)), [Daniel526](https://github.com/code-423n4/2023-11-kelp-findings/issues/356), [Shaheen](https://github.com/code-423n4/2023-11-kelp-findings/issues/230), [PENGUN](https://github.com/code-423n4/2023-11-kelp-findings/issues/209), [glcanvas](https://github.com/code-423n4/2023-11-kelp-findings/issues/195), [0xbrett8571](https://github.com/code-423n4/2023-11-kelp-findings/issues/158), [0xmystery](https://github.com/code-423n4/2023-11-kelp-findings/issues/101), and Bauer ([1](https://github.com/code-423n4/2023-11-kelp-findings/issues/48), [2](https://github.com/code-423n4/2023-11-kelp-findings/issues/39))*

The [depositAsset](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTDepositPool.sol#L119C14-L119C26) function of the [LRTDepositPool](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTDepositPool.sol#L19C10-L19C24) contract, enables users to deposit assets into the protocol, getting [RSETH](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/RSETH.sol#L14C10-L14C15) tokens in return. The function doesn't have any type of slippage control; this is relevant in the context of the [depositAsset](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTDepositPool.sol#L119C14-L119C26) function, since the amount of tokens received by the user is determined by an interaction with an oracle, meaning that the amount received in return may vary indefinitely while the request is waiting to be executed.

*Also the users will have no defense against price manipulations attacks, if they where to be found after the protocol's deployment.*

### Proof of Concept

As can be observed by looking at its parameters and implementation, the [depositAsset](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTDepositPool.sol#L119C14-L119C26) function of the [LRTDepositPool](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTDepositPool.sol#L19C10-L19C24) contract, doesn't have any type of slippage protection:

        function depositAsset(
            address asset,
            uint256 depositAmount
        )
            external
            whenNotPaused
            nonReentrant
            onlySupportedAsset(asset)
        {
            // checks
            if (depositAmount == 0) {
                revert InvalidAmount();
            }
            if (depositAmount > getAssetCurrentLimit(asset)) {
                revert MaximumDepositLimitReached();
            }

            if (!IERC20(asset).transferFrom(msg.sender, address(this), depositAmount)) {
                revert TokenTransferFailed();
            }

            // interactions
            uint256 rsethAmountMinted = _mintRsETH(asset, depositAmount);

            emit AssetDeposit(asset, depositAmount, rsethAmountMinted);
        }

Meaning that users have no control over how many [RSETH](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/RSETH.sol#L14C10-L14C15) tokens they will get in return for depositing in the contract.

The amount of tokens to be minted, with respect to the deposited amount, is determined by the [getRsETHAmountToMint](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTDepositPool.sol#L95) function of the same contract (inside of [\_mintRsETH](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTDepositPool.sol#L151)):

        function getRsETHAmountToMint(
            address asset,
            uint256 amount
        )
            public
            view
            override
            returns (uint256 rsethAmountToMint)
        {
            // setup oracle contract
            address lrtOracleAddress = lrtConfig.getContract(LRTConstants.LRT_ORACLE);
            ILRTOracle lrtOracle = ILRTOracle(lrtOracleAddress);

            // calculate rseth amount to mint based on asset amount and asset exchange rate
            rsethAmountToMint = (amount * lrtOracle.getAssetPrice(asset)) / lrtOracle.getRSETHPrice();
        }

As can be observed, this function uses two oracle interactions to determine how many tokens to mint, [getAssetPrice](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTOracle.sol#L45) and [getRSETHPrice](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTOracle.sol#L52) (with [getRSETHPrice](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTOracle.sol#L52) internally calling [getAssetPrice](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTOracle.sol#L45) as well).

The [getAssetPrice](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTOracle.sol#L45) function queries the external oracle for the asset price:

        function getAssetPrice(address asset) public view onlySupportedAsset(asset) returns (uint256) {
            return IPriceFetcher(assetPriceOracle[asset]).getAssetPrice(asset);
        }

Meaning that the user has no way to predict how many [RSETH](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/RSETH.sol#L14C10-L14C15) they will get back at the moment of minting, as the price could be updated while the request is in the mempool.

Here is a very simple foundry script that shows that an oracle price modification, can largely impact the amount of tokens received in return by the user (implying that the [depositAsset](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTDepositPool.sol#L119C14-L119C26) function has no protection against it).

<details>

Place it in the `test` folder to preserve dependencies:

    // SPDX-License-Identifier: UNLICENSED

    pragma solidity 0.8.21;

    import { BaseTest } from "./BaseTest.t.sol";
    import { LRTDepositPool } from "src/LRTDepositPool.sol";
    import { RSETHTest, ILRTConfig, UtilLib, LRTConstants } from "./RSETHTest.t.sol";
    import { ILRTDepositPool } from "src/interfaces/ILRTDepositPool.sol";

    import { TransparentUpgradeableProxy } from "@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol";
    import { ProxyAdmin } from "@openzeppelin/contracts/proxy/transparent/ProxyAdmin.sol";

    import "forge-std/console.sol";

    contract LRTOracleMock {

        uint256 assetPrice = 1e18;

        function setAssetPrice(uint256 newPrice) external {
            assetPrice = newPrice;
        }

        function getAssetPrice(address) external view returns (uint256) {
            return assetPrice;
        }

        function getRSETHPrice() external pure returns (uint256) {
            return 1e18;
        }
    }

    contract MockNodeDelegator {
        function getAssetBalance(address) external pure returns (uint256) {
            return 1e18;
        }
    }

    contract LRTDepositPoolTest is BaseTest, RSETHTest {
        LRTDepositPool public lrtDepositPool;
        LRTOracleMock public mockOracle;

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
            mockOracle = new LRTOracleMock();

            // initialize RSETH. LRTCOnfig is already initialized in RSETHTest
            rseth.initialize(address(admin), address(lrtConfig));
            vm.startPrank(admin);
            // add rsETH to LRT config
            lrtConfig.setRSETH(address(rseth));
            // add oracle to LRT config
            lrtConfig.setContract(LRTConstants.LRT_ORACLE, address(mockOracle));

            // add minter role for rseth to lrtDepositPool
            rseth.grantRole(rseth.MINTER_ROLE(), address(lrtDepositPool));

            vm.stopPrank();
        }
    }

    contract LRTDepositPoolDepositAsset is LRTDepositPoolTest {
        address public rETHAddress;

        function setUp() public override {
            super.setUp();

            // initialize LRTDepositPool
            lrtDepositPool.initialize(address(lrtConfig));

            rETHAddress = address(rETH);

            // add manager role within LRTConfig
            vm.startPrank(admin);
            lrtConfig.grantRole(LRTConstants.MANAGER, manager);
            vm.stopPrank();
        }


        function test_OracleCanModifyAssetMinted() external {
            
            vm.prank(alice);
            rETH.approve(address(lrtDepositPool), 2 ether);
            vm.prank(bob);
            rETH.approve(address(lrtDepositPool), 2 ether); 

            uint256 aliceDepositTime = block.timestamp;
            vm.prank(alice);
            lrtDepositPool.depositAsset(rETHAddress, 2 ether);

            mockOracle.setAssetPrice(1e17);

            uint256 bobDepositTime = block.timestamp;
            vm.prank(bob);
            lrtDepositPool.depositAsset(rETHAddress, 2 ether);

            uint256 aliceBalanceAfter = rseth.balanceOf(address(alice));
            uint256 bobBalanceAfter = rseth.balanceOf(address(bob));
            
            console.log(aliceBalanceAfter);
            console.log(bobBalanceAfter);    

            console.log(aliceDepositTime);
            console.log(bobDepositTime); 

            assertEq(aliceDepositTime, bobDepositTime);
            assertGt(aliceBalanceAfter, bobBalanceAfter);

        }

    }

</details>

### Recommended Mitigation Steps

An additional parameter could be added to the [depositAsset](https://github.com/code-423n4/2023-11-kelp/blob/c5fdc2e62c5e1d78769f44d6e34a6fb9e40c00f0/src/LRTDepositPool.sol#L119C14-L119C26) function, to let users decide the minimum amount of tokens to be received, with a relative check after minting.

**[gus (Kelp) confirmed](https://github.com/code-423n4/2023-11-kelp-findings/issues/148#issuecomment-1825369647)**

**[manoj9april (Kelp) disagreed with severity and commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/148#issuecomment-1837411885):**
 > Slippage control makes more sense in DEXes. But in staking protocols, where prices are mostly stable and users can reject if it txn takes longer time.<br>
> But it is a useful recommendation.<br>
> I think we can move it to QA.

**[0xDjango (judge) commented](https://github.com/code-423n4/2023-11-kelp-findings/issues/148#issuecomment-1839011720):**
 > Sticking with medium. There is no protection for the user in the current implementation. `minShares` checks are commonly implemented in staking contracts.



***

# Low Risk and Non-Critical Issues

For this audit, 125 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2023-11-kelp-findings/issues/581) by **m\_Rassska** received the top score from the judge.

*The following wardens also submitted reports: [CatsSecurity](https://github.com/code-423n4/2023-11-kelp-findings/issues/697), [Pechenite](https://github.com/code-423n4/2023-11-kelp-findings/issues/650), [hals](https://github.com/code-423n4/2023-11-kelp-findings/issues/415), [0xepley](https://github.com/code-423n4/2023-11-kelp-findings/issues/881), [0xffchain](https://github.com/code-423n4/2023-11-kelp-findings/issues/878), [btk](https://github.com/code-423n4/2023-11-kelp-findings/issues/875), [hunter\_w3b](https://github.com/code-423n4/2023-11-kelp-findings/issues/873), [turvy\_fuzz](https://github.com/code-423n4/2023-11-kelp-findings/issues/859), [adriro](https://github.com/code-423n4/2023-11-kelp-findings/issues/840), [anarcheuz](https://github.com/code-423n4/2023-11-kelp-findings/issues/831), [Madalad](https://github.com/code-423n4/2023-11-kelp-findings/issues/815), [spark](https://github.com/code-423n4/2023-11-kelp-findings/issues/803), [ge6a](https://github.com/code-423n4/2023-11-kelp-findings/issues/798), [shealtielanz](https://github.com/code-423n4/2023-11-kelp-findings/issues/783), [\_thanos1](https://github.com/code-423n4/2023-11-kelp-findings/issues/774), [0xAadi](https://github.com/code-423n4/2023-11-kelp-findings/issues/769), [phoenixV110](https://github.com/code-423n4/2023-11-kelp-findings/issues/757), [rouhsamad](https://github.com/code-423n4/2023-11-kelp-findings/issues/755), [poneta](https://github.com/code-423n4/2023-11-kelp-findings/issues/745), [0xvj](https://github.com/code-423n4/2023-11-kelp-findings/issues/724), [Udsen](https://github.com/code-423n4/2023-11-kelp-findings/issues/723), [SandNallani](https://github.com/code-423n4/2023-11-kelp-findings/issues/716), [gesha17](https://github.com/code-423n4/2023-11-kelp-findings/issues/714), [Topmark](https://github.com/code-423n4/2023-11-kelp-findings/issues/710), [dipp](https://github.com/code-423n4/2023-11-kelp-findings/issues/708), [adeolu](https://github.com/code-423n4/2023-11-kelp-findings/issues/705), [leegh](https://github.com/code-423n4/2023-11-kelp-findings/issues/686), [tallo](https://github.com/code-423n4/2023-11-kelp-findings/issues/685), [osmanozdemir1](https://github.com/code-423n4/2023-11-kelp-findings/issues/677), [cartlex\_](https://github.com/code-423n4/2023-11-kelp-findings/issues/673), [RaoulSchaffranek](https://github.com/code-423n4/2023-11-kelp-findings/issues/667), [Matin](https://github.com/code-423n4/2023-11-kelp-findings/issues/648), [ZanyBonzy](https://github.com/code-423n4/2023-11-kelp-findings/issues/640), [cheatc0d3](https://github.com/code-423n4/2023-11-kelp-findings/issues/637), [Tumelo\_Crypto](https://github.com/code-423n4/2023-11-kelp-findings/issues/635), [GREY-HAWK-REACH](https://github.com/code-423n4/2023-11-kelp-findings/issues/626), [twcctop](https://github.com/code-423n4/2023-11-kelp-findings/issues/617), [bronze\_pickaxe](https://github.com/code-423n4/2023-11-kelp-findings/issues/616), [Yanchuan](https://github.com/code-423n4/2023-11-kelp-findings/issues/579), [gumgumzum](https://github.com/code-423n4/2023-11-kelp-findings/issues/578), [0xHelium](https://github.com/code-423n4/2023-11-kelp-findings/issues/577), [Amithuddar](https://github.com/code-423n4/2023-11-kelp-findings/issues/575), [Inspecktor](https://github.com/code-423n4/2023-11-kelp-findings/issues/569), [sakshamguruji](https://github.com/code-423n4/2023-11-kelp-findings/issues/568), [jasonxiale](https://github.com/code-423n4/2023-11-kelp-findings/issues/566), [joaovwfreire](https://github.com/code-423n4/2023-11-kelp-findings/issues/545), [0xblackskull](https://github.com/code-423n4/2023-11-kelp-findings/issues/541), [0xLeveler](https://github.com/code-423n4/2023-11-kelp-findings/issues/539), [baice](https://github.com/code-423n4/2023-11-kelp-findings/issues/535), [SBSecurity](https://github.com/code-423n4/2023-11-kelp-findings/issues/534), [xAriextz](https://github.com/code-423n4/2023-11-kelp-findings/issues/526), [niser93](https://github.com/code-423n4/2023-11-kelp-findings/issues/525), [ro1sharkm](https://github.com/code-423n4/2023-11-kelp-findings/issues/502), [hihen](https://github.com/code-423n4/2023-11-kelp-findings/issues/500), [Phantasmagoria](https://github.com/code-423n4/2023-11-kelp-findings/issues/499), [McToady](https://github.com/code-423n4/2023-11-kelp-findings/issues/485), [bareli](https://github.com/code-423n4/2023-11-kelp-findings/issues/483), [0x1337](https://github.com/code-423n4/2023-11-kelp-findings/issues/482), [Aamir](https://github.com/code-423n4/2023-11-kelp-findings/issues/479), [almurhasan](https://github.com/code-423n4/2023-11-kelp-findings/issues/473), [eeshenggoh](https://github.com/code-423n4/2023-11-kelp-findings/issues/458), [seerether](https://github.com/code-423n4/2023-11-kelp-findings/issues/453), [zhaojie](https://github.com/code-423n4/2023-11-kelp-findings/issues/447), [Bauchibred](https://github.com/code-423n4/2023-11-kelp-findings/issues/439), [Soul22](https://github.com/code-423n4/2023-11-kelp-findings/issues/432), [ayden](https://github.com/code-423n4/2023-11-kelp-findings/issues/431), [lsaudit](https://github.com/code-423n4/2023-11-kelp-findings/issues/423), [deepkin](https://github.com/code-423n4/2023-11-kelp-findings/issues/399), [chaduke](https://github.com/code-423n4/2023-11-kelp-findings/issues/394), [AerialRaider](https://github.com/code-423n4/2023-11-kelp-findings/issues/392), [Shaheen](https://github.com/code-423n4/2023-11-kelp-findings/issues/383), [Cryptor](https://github.com/code-423n4/2023-11-kelp-findings/issues/382), [boredpukar](https://github.com/code-423n4/2023-11-kelp-findings/issues/381), [TeamSS](https://github.com/code-423n4/2023-11-kelp-findings/issues/380), [alexfilippov314](https://github.com/code-423n4/2023-11-kelp-findings/issues/376), [Draiakoo](https://github.com/code-423n4/2023-11-kelp-findings/issues/369), [Daniel526](https://github.com/code-423n4/2023-11-kelp-findings/issues/361), [amaechieth](https://github.com/code-423n4/2023-11-kelp-findings/issues/357), [zach](https://github.com/code-423n4/2023-11-kelp-findings/issues/351), [squeaky\_cactus](https://github.com/code-423n4/2023-11-kelp-findings/issues/348), [crack-the-kelp](https://github.com/code-423n4/2023-11-kelp-findings/issues/341), [King\_](https://github.com/code-423n4/2023-11-kelp-findings/issues/336), [MatricksDeCoder](https://github.com/code-423n4/2023-11-kelp-findings/issues/326), [Noro](https://github.com/code-423n4/2023-11-kelp-findings/issues/320), [circlelooper](https://github.com/code-423n4/2023-11-kelp-findings/issues/313), [wisdomn\_](https://github.com/code-423n4/2023-11-kelp-findings/issues/310), [0xrugpull\_detector](https://github.com/code-423n4/2023-11-kelp-findings/issues/306), [pep7siup](https://github.com/code-423n4/2023-11-kelp-findings/issues/298), [Juntao](https://github.com/code-423n4/2023-11-kelp-findings/issues/294), [paritomarrr](https://github.com/code-423n4/2023-11-kelp-findings/issues/292), [soliditytaker](https://github.com/code-423n4/2023-11-kelp-findings/issues/285), [ziyou-](https://github.com/code-423n4/2023-11-kelp-findings/issues/272), [codynhat](https://github.com/code-423n4/2023-11-kelp-findings/issues/270), [passion](https://github.com/code-423n4/2023-11-kelp-findings/issues/262), [catellatech](https://github.com/code-423n4/2023-11-kelp-findings/issues/261), [supersizer0x](https://github.com/code-423n4/2023-11-kelp-findings/issues/251), [stackachu](https://github.com/code-423n4/2023-11-kelp-findings/issues/248), [evmboi32](https://github.com/code-423n4/2023-11-kelp-findings/issues/245), [TheSchnilch](https://github.com/code-423n4/2023-11-kelp-findings/issues/243), [taner2344](https://github.com/code-423n4/2023-11-kelp-findings/issues/239), [glcanvas](https://github.com/code-423n4/2023-11-kelp-findings/issues/216), [marchev](https://github.com/code-423n4/2023-11-kelp-findings/issues/215), [Stormreckson](https://github.com/code-423n4/2023-11-kelp-findings/issues/212), [rvierdiiev](https://github.com/code-423n4/2023-11-kelp-findings/issues/208), [Eigenvectors](https://github.com/code-423n4/2023-11-kelp-findings/issues/207), [PENGUN](https://github.com/code-423n4/2023-11-kelp-findings/issues/196), [ElCid](https://github.com/code-423n4/2023-11-kelp-findings/issues/178), [ke1caM](https://github.com/code-423n4/2023-11-kelp-findings/issues/171), [0xluckhu](https://github.com/code-423n4/2023-11-kelp-findings/issues/168), [desaperh](https://github.com/code-423n4/2023-11-kelp-findings/issues/156), [debo](https://github.com/code-423n4/2023-11-kelp-findings/issues/153), [merlinboii](https://github.com/code-423n4/2023-11-kelp-findings/issues/149), [ABAIKUNANBAEV](https://github.com/code-423n4/2023-11-kelp-findings/issues/140), [pipidu83](https://github.com/code-423n4/2023-11-kelp-findings/issues/116), [LinKenji](https://github.com/code-423n4/2023-11-kelp-findings/issues/112), [0xbrett8571](https://github.com/code-423n4/2023-11-kelp-findings/issues/109), [0xmystery](https://github.com/code-423n4/2023-11-kelp-findings/issues/102), [zhaojohnson](https://github.com/code-423n4/2023-11-kelp-findings/issues/96), [critical-or-high](https://github.com/code-423n4/2023-11-kelp-findings/issues/88), [MaslarovK](https://github.com/code-423n4/2023-11-kelp-findings/issues/73), [T1MOH](https://github.com/code-423n4/2023-11-kelp-findings/issues/72), [ubl4nk](https://github.com/code-423n4/2023-11-kelp-findings/issues/51), [Bauer](https://github.com/code-423n4/2023-11-kelp-findings/issues/46), and [Tadev](https://github.com/code-423n4/2023-11-kelp-findings/issues/36).*

## Summary

### Low Severity
  * **[L-01] Prevent deposits to the nodes with max amount of assets being staked already**
  * **[L-02] Set the price feeds during an initialization**
  * **[L-03] Dropping a dust to grief the staking process**
  * **[L-04] Approve to `StrategyManager` during an initialization**
  * **[L-05] Enforce `maxnodeDelegatorCount > nodeDelegatorQueue.length`**
  * **[L-06] Place an assertion to ensure the system in paused/unpaused state**

### Non-Critical Severity
  * **[N-01] Introduce `removeNodeDelegatorToQueue`**

## [L-01] Prevent deposits to the nodes with max amount of assets being staked already

If the node has staked already 1e23 of assets, it should not receive any transfers, since there is no more space to stake at EigenLayer.

### Example of an occurrence

Could be implied [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L183-L197).

## [L-02] Set the price feeds during an initialization

Currently, the price feeds are set by using `updatePriceOracleFor()` after an `LRTOPracle` being deployed. However, it's better to set up everything during an initialization process.

### Example of an occurrence

Could be added, for instance, [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L29-L35).

## [L-03] Dropping a dust to grief the staking process
After an amount of lst being transferred to a specified `NodeDelegator`, the stake on EigenLayer is expected to happen. However, if there is a possibility to grief `depositAssetIntoStrategy`, adversary might drop some dust and front-run the tx submitted by `LRTManager`. This forces to transfer some assets back to the pool in order to successfully stake at EigenLayer.
  
### Example of an occurrence

It's better to provide an opportunity for `LRTManager` to decide, how much lst amount is about to be transferred and not blindly rely on the total balance [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L63).

## [L-04] Approve to `StrategyManager` during an initialization
There is no need to inflate the bytecode for `NodeDelegator` by introducing a special function for an approval grants. Instead, approve to the `eigenlayerStrategyManagerAddress` during an initialization.
  
### Example of an occurrence

Could be added, for instance, [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol#L29-L35).

## [L-05] Enforce `maxnodeDelegatorCount > nodeDelegatorQueue.length`
Put an assertion to ensure that the `maxnodeDelegatorCount > nodeDelegatorQueue.length` to prevent unexpected behavior from happening. 
  
### Example of an occurrence

Could be added, for instance, [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L202-L205).

## [L-06] Place an assertion to ensure the system in paused/unpaused state
The functions `pause()/unpause()` are super important in case something suspicous starts to happen. An `LRTManager` might accidentally invoke `unpause()` instead of hitting `pause()` and convince himself of stopping suspicous activity. However, this might give an attacker the chance to continue his dirty activity.
  
### Example of an occurrence

Could be added, for instance, [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L208-L215).

## [N-01] Introduce `removeNodeDelegatorFromQueue`
It is recommended to introduce `removeNodeDelegatorFromQueue` in case the `NodeDelegator` is no longer needed.

### Example of an occurrence

Could be introduced, for instance, [here](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L158).



***

# Gas Optimizations

For this audit, 16 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2023-11-kelp-findings/issues/591) by **0xVolcano** received the top score from the judge.

*The following wardens also submitted reports: [hunter\_w3b](https://github.com/code-423n4/2023-11-kelp-findings/issues/715), [unique](https://github.com/code-423n4/2023-11-kelp-findings/issues/583), [lsaudit](https://github.com/code-423n4/2023-11-kelp-findings/issues/424), [tala7985](https://github.com/code-423n4/2023-11-kelp-findings/issues/849), [Raihan](https://github.com/code-423n4/2023-11-kelp-findings/issues/813), [0xAnah](https://github.com/code-423n4/2023-11-kelp-findings/issues/804), [JCK](https://github.com/code-423n4/2023-11-kelp-findings/issues/802), [0xhex](https://github.com/code-423n4/2023-11-kelp-findings/issues/731), [0xta](https://github.com/code-423n4/2023-11-kelp-findings/issues/725), [ZanyBonzy](https://github.com/code-423n4/2023-11-kelp-findings/issues/706), [shamsulhaq123](https://github.com/code-423n4/2023-11-kelp-findings/issues/629), [Sathish9098](https://github.com/code-423n4/2023-11-kelp-findings/issues/491), [ABAIKUNANBAEV](https://github.com/code-423n4/2023-11-kelp-findings/issues/144), [Rickard](https://github.com/code-423n4/2023-11-kelp-findings/issues/137), and [rumen](https://github.com/code-423n4/2023-11-kelp-findings/issues/75).*

## [G-01] Avoid zero to non-zero storage writes

When a storage variable goes from zero to non-zero, the user must pay 22,100 gas total (20,000 gas for a zero to non-zero write and 2,100 for a cold storage access).

This is why the Openzeppelin reentrancy guard registers functions as active or not with 1 and 2 rather than 0 and 1. It only costs 5,000 gas to alter a storage variable from non-zero to non-zero.

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L19-L38
```solidity
File: /src/LRTDepositPool.sol
19:contract LRTDepositPool is ILRTDepositPool, LRTConfigRoleChecker, PausableUpgradeable, ReentrancyGuardUpgradeable {
20:    uint256 public maxNodeDelegatorCount;

29:    /// @dev Initializes the contract
30:    /// @param lrtConfigAddr LRT config address
31:    function initialize(address lrtConfigAddr) external initializer {
32:        UtilLib.checkNonZeroAddress(lrtConfigAddr);
33:        __Pausable_init();
34:        __ReentrancyGuard_init();
35:        maxNodeDelegatorCount = 10;
36:        lrtConfig = ILRTConfig(lrtConfigAddr);
37:        emit UpdatedLRTConfig(lrtConfigAddr);
38:    }
```

When declaring the variable `maxNodDelegatorCount` the default value is applied since we do not assign anything, the default value of `uint256` is `0`.

We then call the function `initialize()` and here we assign the value `10` `maxNodeDelegatorCount = 10;`. This means our value is updated from `0`  to `10`.

Since the `initialize()` function will always be called first, we can initialize our variable `maxNodDelegatorCount`  with `1` during declaration which should avoid the zero to non zero writes.

```diff
 /// @title LRTDepositPool - Deposit Pool Contract for LSTs
 /// @notice Handles LST asset deposits
 contract LRTDepositPool is ILRTDepositPool, LRTConfigRoleChecker, PausableUpgradeable, ReentrancyGuardUpgradeable {
-    uint256 public maxNodeDelegatorCount;
+    uint256 public maxNodeDelegatorCount = 1;

     address[] public nodeDelegatorQueue;

```

## [G-02] We can optimize the function `updateLRTConfig()`

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/utils/LRTConfigRoleChecker.sol#L47-L51
```solidity
File: /src/utils/LRTConfigRoleChecker.sol
47:    function updateLRTConfig(address lrtConfigAddr) external virtual onlyLRTAdmin {
48:        UtilLib.checkNonZeroAddress(lrtConfigAddr);
49:        lrtConfig = ILRTConfig(lrtConfigAddr);
50:        emit UpdatedLRTConfig(lrtConfigAddr);
51:    }
```

The above function uses the modifier `onlyLRTAdmin` which has the following implementation:
```solidity
    modifier onlyLRTAdmin() {
        bytes32 DEFAULT_ADMIN_ROLE = 0x00;
        if (!IAccessControl(address(lrtConfig)).hasRole(DEFAULT_ADMIN_ROLE, msg.sender)) {
            revert ILRTConfig.CallerNotLRTConfigAdmin();
        }
        _;
    }
```

The modifier reads a state variable `lrtConfig` and also makes an external call to `hasRole()`.

If the check does not pass we have a revert. Using this modifier in our function means we run the check first and revert if the check does not pass.

If the check passes, our function has one more check `UtilLib.checkNonZeroAddress` which basically checks that our address is not zero. This is way cheaper compared to the check inside the modifier

If this check does not pass, it means we would also revert, now suppose the first check(modifier) passes but the non zero fails, the gas consumed inside the modifier doing the state read and external call would all be wasted.

We can do this check more efficiently by prioritizing cheaper checks first but for this to work, we would need to inline our modifier as shown below:

```diff
diff --git a/src/utils/LRTConfigRoleChecker.sol b/src/utils/LRTConfigRoleChecker.sol
index 991d0e9..1b9917f 100644
--- a/src/utils/LRTConfigRoleChecker.sol
+++ b/src/utils/LRTConfigRoleChecker.sol
@@ -44,8 +44,12 @@ abstract contract LRTConfigRoleChecker {
     /// @notice Updates the LRT config contract
     /// @dev only callable by LRT admin
     /// @param lrtConfigAddr the new LRT config contract Address
-    function updateLRTConfig(address lrtConfigAddr) external virtual onlyLRTAdmin {
+    function updateLRTConfig(address lrtConfigAddr) external virtual  {
         UtilLib.checkNonZeroAddress(lrtConfigAddr);
+        bytes32 DEFAULT_ADMIN_ROLE = 0x00;
+        if (!IAccessControl(address(lrtConfig)).hasRole(DEFAULT_ADMIN_ROLE, msg.sender)) {
+            revert ILRTConfig.CallerNotLRTConfigAdmin();
+        }
         lrtConfig = ILRTConfig(lrtConfigAddr);
         emit UpdatedLRTConfig(lrtConfigAddr);
     }
     
```

## [G-03] Emit local variables instead of state variables(not found by bot)

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L202-L205
```solidity
File: /src/LRTDepositPool.sol
202:    function updateMaxNodeDelegatorCount(uint256 maxNodeDelegatorCount_) external onlyLRTAdmin {
203:        maxNodeDelegatorCount = maxNodeDelegatorCount_;
204:        emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount);
205:    }
```

```diff
@@ -201,7 +201,7 @@ contract LRTDepositPool is ILRTDepositPool, LRTConfigRoleChecker, PausableUpgrad
     /// @param maxNodeDelegatorCount_ Maximum count of node delegator
     function updateMaxNodeDelegatorCount(uint256 maxNodeDelegatorCount_) external onlyLRTAdmin {
         maxNodeDelegatorCount = maxNodeDelegatorCount_;
-        emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount);
+        emit MaxNodeDelegatorCountUpdated(maxNodeDelegatorCount_);//@audit emit the local var
     }
```

## [G-04] Don't cache calls/variables if using them once

### No need to cache `lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER)`

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L38-L46
```solidity
File: /src/NodeDelegator.sol
38:    function maxApproveToEigenStrategyManager(address asset)
39:        external
40:        override
41:        onlySupportedAsset(asset)
42:        onlyLRTManager
43:    {
44:        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);
45:        IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);
46:    }
```

The variable `eigenlayerStrategyManagerAddress` is only used once, therefore no need to cache the call.

```diff
diff --git a/src/NodeDelegator.sol b/src/NodeDelegator.sol
index bba20ca..989218c 100644
--- a/src/NodeDelegator.sol
+++ b/src/NodeDelegator.sol
@@ -41,8 +41,7 @@ contract NodeDelegator is INodeDelegator, LRTConfigRoleChecker, PausableUpgradea
         onlySupportedAsset(asset)
         onlyLRTManager
     {
-        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);
-        IERC20(asset).approve(eigenlayerStrategyManagerAddress, type(uint256).max);
+        IERC20(asset).approve(lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY
```

### No need to cache `lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER)`

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L51-L68

```solidity
File: /src/NodeDelegator.sol
51:    function depositAssetIntoStrategy(address asset)

58:    {
59:        address strategy = lrtConfig.assetStrategy(asset);
60:        IERC20 token = IERC20(asset);
61:        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

63:        uint256 balance = token.balanceOf(address(this));

65:        emit AssetDepositIntoStrategy(asset, strategy, balance);

67:IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
68:    }
```

```diff
     /// @notice Deposits an asset lying in this NDC into its strategy
@@ -58,13 +58,12 @@ contract NodeDelegator is INodeDelegator, LRTConfigRoleChecker, PausableUpgradea
     {
         address strategy = lrtConfig.assetStrategy(asset);
         IERC20 token = IERC20(asset);
-        address eigenlayerStrategyManagerAddress = lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER);

         uint256 balance = token.balanceOf(address(this));

         emit AssetDepositIntoStrategy(asset, strategy, balance);

-        IEigenStrategyManager(eigenlayerStrategyManagerAddress).depositIntoStrategy(IStrategy(strategy), token, balance);
+        IEigenStrategyManager(lrtConfig.getContract(LRTConstants.EIGEN_STRATEGY_MANAGER)).depositIntoStrategy(IStrategy(strategy), token, balance);
     }

```

### No need to cache `lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL)`

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L74-L89


```solidity
File: /src/NodeDelegator.sol
74:    function transferBackToLRTDepositPool(

84:        address lrtDepositPool = lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL);

86:        if (!IERC20(asset).transfer(lrtDepositPool, amount)) {
87:            revert TokenTransferFailed();
88:        }
89:    }
```

```diff
     /// @notice Deposits an asset lying in this NDC into its strategy
@@ -81,9 +81,8 @@ contract NodeDelegator is INodeDelegator, LRTConfigRoleChecker, PausableUpgradea
         onlySupportedAsset(asset)
         onlyLRTManager
     {
-        address lrtDepositPool = lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL);

-        if (!IERC20(asset).transfer(lrtDepositPool, amount)) {
+        if (!IERC20(asset).transfer(lrtConfig.getContract(LRTConstants.LRT_DEPOSIT_POOL), amount)) {
             revert TokenTransferFailed();
         }
     }
```

### No need to cache `lrtConfig.assetStrategy(asset)`

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L121-L124

```solidity
File: /src/NodeDelegator.sol
121:    function getAssetBalance(address asset) external view override returns (uint256) {
122:        address strategy = lrtConfig.assetStrategy(asset);
123:        return IStrategy(strategy).userUnderlyingView(address(this));
124:    }
```

```diff
     function getAssetBalance(address asset) external view override returns (uin
t256) {
-        address strategy = lrtConfig.assetStrategy(asset);
-        return IStrategy(strategy).userUnderlyingView(address(this));
+        return IStrategy(lrtConfig.assetStrategy(asset)).userUnderlyingView(address(this));
     }
```

### No need to cache `nodeDelegatorQueue[ndcIndex]`

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L183-L197

```solidity
File: /src/LRTDepositPool.sol
183:    function transferAssetToNodeDelegator(

192:    {
193:        address nodeDelegator = nodeDelegatorQueue[ndcIndex];
194:        if (!IERC20(asset).transfer(nodeDelegator, amount)) {
195:            revert TokenTransferFailed();
196:        }
197:    }
```

```diff
@@ -190,8 +190,7 @@ contract LRTDepositPool is ILRTDepositPool, LRTConfigRoleChecker, PausableUpgrad
         onlyLRTManager
         onlySupportedAsset(asset)
     {
-        address nodeDelegator = nodeDelegatorQueue[ndcIndex];
-        if (!IERC20(asset).transfer(nodeDelegator, amount)) {
+        if (!IERC20(asset).transfer(nodeDelegatorQueue[ndcIndex], amount)) {
             revert TokenTransferFailed();
         }
     }
```

## [G-05] Caching the length of calldata increases gas cost instead of reducing it

https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L162-L177
```solidity
File: /src/LRTDepositPool.sol
162:    function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {
163:        uint256 length = nodeDelegatorContracts.length;
164:        if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
165:            revert MaximumNodeDelegatorCountReached();
166:        }

168:        for (uint256 i; i < length;) {
169:            UtilLib.checkNonZeroAddress(nodeDelegatorContracts[i]);
170:            nodeDelegatorQueue.push(nodeDelegatorContracts[i]);
171:            emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]);
172:            unchecked {
173:                ++i;
174:            }
175:        }
176:    }
```

```diff
     function addNodeDelegatorContractToQueue(address[] calldata nodeDelegatorContracts) external onlyLRTAdmin {
         uint256 length = nodeDelegatorContracts.length;
-        if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
+        if (nodeDelegatorQueue.length + nodeDelegatorContracts.length > maxNodeDelegatorCount) {
             revert MaximumNodeDelegatorCountReached();
         }

-        for (uint256 i; i < length;) {
+        for (uint256 i; i < nodeDelegatorContracts.length;) {
```



***


# Audit Analysis

For this audit, 21 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2023-11-kelp-findings/issues/618) by **CatsSecurity** received the top score from the judge.

*The following wardens also submitted reports: [SBSecurity](https://github.com/code-423n4/2023-11-kelp-findings/issues/876), [hunter\_w3b](https://github.com/code-423n4/2023-11-kelp-findings/issues/867), [albahaca](https://github.com/code-423n4/2023-11-kelp-findings/issues/806), [sakshamguruji](https://github.com/code-423n4/2023-11-kelp-findings/issues/657), [foxb868](https://github.com/code-423n4/2023-11-kelp-findings/issues/546), [0xbrett8571](https://github.com/code-423n4/2023-11-kelp-findings/issues/508), [Bauchibred](https://github.com/code-423n4/2023-11-kelp-findings/issues/440), [invitedtea](https://github.com/code-423n4/2023-11-kelp-findings/issues/437), [0xepley](https://github.com/code-423n4/2023-11-kelp-findings/issues/378), [0xSmartContract](https://github.com/code-423n4/2023-11-kelp-findings/issues/872), [fouzantanveer](https://github.com/code-423n4/2023-11-kelp-findings/issues/841), [SAAJ](https://github.com/code-423n4/2023-11-kelp-findings/issues/832), [clara](https://github.com/code-423n4/2023-11-kelp-findings/issues/826), [Sathish9098](https://github.com/code-423n4/2023-11-kelp-findings/issues/720), [unique](https://github.com/code-423n4/2023-11-kelp-findings/issues/671), [KeyKiril](https://github.com/code-423n4/2023-11-kelp-findings/issues/586), [Myd](https://github.com/code-423n4/2023-11-kelp-findings/issues/574), [digitizeworx](https://github.com/code-423n4/2023-11-kelp-findings/issues/512), [ZanyBonzy](https://github.com/code-423n4/2023-11-kelp-findings/issues/404), and [glcanvas](https://github.com/code-423n4/2023-11-kelp-findings/issues/250).*

### Description of Kelp
Kelp represents a collaborative DAO with the purpose of enhancing liquidity, DeFi, and maximizing rewards for assets that are restaked. The core of this initiative lies in the integration of a single liquid restaked token made for recognized LSTs, ensuring seamless accessibility. The protocol will allow users to employ their rsETH within EigenLayer strategies, enabling them to generate returns. The ongoing development of rsETH will be carried out within the framework of the Kelp brand.

### 1. System Overview

**1.1 Scoping**

- [LRTConfig](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol): This contract includes the functionality to configure the system such as adding new supported assets, updating assets' deposit limit, updating an asset's strategy, setting the `rsETH` contract address, setting the token address and numerous getter functions.
- [LRTDepositPool](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol): This contract is the main user-facing contract and includes the functionality to deposit assets, mint `rsETH`, add node delegator's to the queue, transfer assets to node delegators, update the max node delegators count and numerous getter functions.
- [NodeDelegator](https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol): This contract is the recipient of funds from the [LRTDepositPool](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol) contract and in turn sends them to the EigenLayer strategies, includes functionality to approve assets to EigenLayer, deposit assets into the strategy, transfer funds back to [LRTDepositPool](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol), and numerous getter functions.
- [LRTOracle](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol): This contract is utilized as the `Oracle` to get prices of liquid staking tokens, includes functionality to update the price oracle for supported assets, as well as fetch supported asset and `rsETH` prices.
- [RSETH](https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol): This contract is the `rsETH` ERC20 Token the protocol uses and contains all the standard functionality as per OpenZeppelin's requirements.
- [ChainlinkPriceOracle](https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol): This contract is the `Chainlink Price Oracle` wrapper to integrate their oracles in the [LRTOracle](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol) and contains all standard Chainlink Oracle functionality. 
- [LRTConfigRoleChecker](https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConfigRoleChecker.sol): This contract is abstract and is responsible for handling role checks such as `onlyLRTManager`, `onlyLRTAdmin`, `onlySupportedAsset`, and can also be used to update the [LRTConfig](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol) contract.
- [UtilLib](https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/UtilLib.sol): This contract is a simple helper library that contains a check for non zero address function.
- [LRTConstants](https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConstants.sol): This contract is contains all the shared constant variables used within the system such as `R_ETH_TOKEN`, `ST_ETH_TOKEN`, `CB_ETH_TOKEN`, `LRT_ORACLE`, `LRT_DEPOSIT_POOL`, `EIGEN_STRATEGY_MANAGER`, `MANAGER`.

**1.2 Access Control Roles**

The contracts uses the following 4 access control modifiers for different mechanisms within the system: 

- `DEFAULT_ADMIN_ROLE`: Used for core administrative functions within the system that have to do with initial setting of addresses as well as updating asset strategies. 
- `onlyLRTADMIN`: Used for functionality within the system that has to do with node delegaotrs and unpausing the contracts.
- `onlyLRTMANAGER`: Used for functionality within the system that has to do with node delegators, transfer of funds between contracts and EigenLayer strategies and price oracle configurations.
- `LRTConstants.MANAGER`: Used for functionality within the system that has to do with supported assets and their deposit limits.

**1.3 Access Restricted Functions**

- `DEFAULT_ADMIN_ROLE`: This role has access to administrative functions such as [updating asset strategies](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L109), [setting the rsETH address](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L144), [setting the token address](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L149) & [setting the contract address](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L165).
- `onlyLRTADMIN`: This role has access to functions related to the Node Delegators such as [adding new node delegator contracts to the queue](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L162), [updating the maximum node delegator count](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L202) and [unpausing](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L213) the various contracts that allow for that.
- `onlyLRTMANAGER`: This role has access to other functions related to the Node Delegators such as [transferring assets from the deposit pool to the node delegator contracts](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L183), as well as [approving assets to the EigenLayer strategy manager](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L38), [depositing assets from node delegators to the strategy](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L51), [transferring funds from node delegators back to the deposit pool](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/NodeDelegator.sol#L74), [updating the price oracle of suported assets](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTOracle.sol#L88), [adding or updating the oracle price feed for assets](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/oracles/ChainlinkPriceOracle.sol#L45), and [pausing](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTDepositPool.sol#L208) the various contracts that allow for that.
- `LRTConstants.MANAGER`: This role has access to functions related to [adding new supported assets](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L73), and [updating the asset deposit limits](https://github.com/code-423n4/2023-11-kelp/blob/f751d7594051c0766c7ecd1e68daeb0661e43ee3/src/LRTConfig.sol#L94).

### 2. Architecture Overview & Diagrams

**2.1 Architecture Overview**

- LRTConfig: The LRTConfig contract is a critical part of the Kelp protocol. It contains the logic for the system's configuration such as supported assets and their deposit limits and asset strategies.

- LRTDepositPool: The LRTDepositPool contract is the main user-facing contract in the protocol responsible for taking in deposits and minting `rsETH` for users. Users can also get information related to current asset limits, total asset deposits and the node delegator queue. Restricted roles have access to node delegator and transferring of funds functions inside this contract.

- LRTOracle: The LRTOracle contract is the one responsible for fetching prices of LSTs (liquid staking tokens). It works together with the LRTDepositPool to maintain accurate price validity and helps in calculating how much `rsETH` to mint for users.

- RSETH: The RSETH Contract is the system's ERC20 token implementation. Not much to say here.

- ChainlinkPriceOracle: The ChainLinkPriceOracle contract is the wrapper contract used to integrate Chainlink's Oracles. It works together with the LRTOracle contract to supply it's fetched data.

- LRTConfigRoleChecker: The LRTConfigRoleChecker contract is the system's implementation for access control modifiers enabling certain functions to only be accessible by privileged roles.

- UtilLib: The UtilLib contract is a helper function lib that contains a check for non zero address function.

- LRTConstants: The LRTConstants contract contains the constant variables shared between the contracts such as `R_ETH_TOKEN` & other supported LSTs, `LRT_ORACLE` & other contracts, as well as the `MANAGER` role.

**2.2 Architecture Diagram**

The architecture diagram visualizes how the contracts in the system interact with eachother and provides short descriptions of the contracts' intended mechanism.

![Architechture - Frame 1](https://user-images.githubusercontent.com/58657666/282805871-37965b8f-cece-4a75-845a-0437ae75d183.jpg)

**2.3 Flow of Funds Diagram**

The flow of funds diagram visualizes the process through which a user goes in order to deposit their assets into the protocol and how they travel to their final destination of the EigenLayer strategies.

![Architechture - Frame 2](https://user-images.githubusercontent.com/58657666/282804967-c299fa9e-ff8b-43ed-9c91-efaee8b950b2.jpg)

**2.4 Core Contracts Parameters Diagram**

The core contracts parameters diagram visualizes the main parameters and/or storage variables in the system and what they represent.

![Architechture - Frame 3](https://user-images.githubusercontent.com/58657666/282804985-134b842b-1bce-4991-981f-83f15c9c7f21.jpg)

### 3. Codebase Assessment
The total nSLOC of the contracts in scope are quite low but it can be seen that the developers have done an excellent job of integrating all the functionality properly. Config, Deposit Pool, Oracles & Node Delegaotrs communicate seamlessly with access control roles functioning as intended wherever needed. The system includes `pause` functionality which, in my opinion, is always a good idea.

- General Redability: The codebase exhibits sound coding practices, upholding readability and organization through well-defined variables and function names. 
- Code Style: The code adheres to a uniform style and indentation pattern, complemented by in-line comments that provide comprehensive guidance throughout it's mechanisms.
- Gas: Gas efficiency is effectively upheld.
- Test Suite: With a test suite coverage of 98%, the protocol did an almost perfect job. The setting up of environments for tests was easy and straight-forward.
- Errors & Events: This is something that codebase lacked in. I didn't come across any handling neither of errors, nor events.
- Upgradability: All core contracts of the protocol maintained upgradabillity, with plans of future expansion & integration this should be a must.

### 4. Level of Documentation
The documentation provided in the github repo was not extensive, although it's understandable for such a small codebase. There was a link to [their blog](https://blog.kelpdao.xyz/reimagine-restaking-with-rseth-developed-by-kelp-78b7a5ae1230) with some additional information on more in-general subjects such as "What is restaking", "What is EigenLayer" and such. That's all good but I would have liked more techinical documentation about the project.

### 5. Systemic & Centralization Risks

**5.1 Systemic Risks**

Although the codebase is small, since this is the protocol's first audit I wouldn't be surprised if a few bugs arise, it is our goal to clear them. I would usually say there is always room for improvement codebase-wise but I belive the developers have done an excellent job here. The owner's have stated that they intend to expand the codebase and conduct subsequent audits for the expansion which is great. Me and my teammate were able to uncover a few vulnerabilities and I am sure other auditors will come up with more stuff.

- Loss of value upon mint: Our team uncovered a vulnerability which if exploited could set the system up in such a way that the 2nd depositor's minting value is significantly reduced. This is quite has quite a serious impact although potentially only possible for the 2nd depositor, even if the system currently has no `withdraw` functionality, it is still a major loss of `value`. The issue arises from the fact that the codebase
 does not use internal accounting during a crucial division to calculate the `rsETH` amount to mint, but instead takes the contracts' balances. This opens the door for an attacker to simply donate/directly send value instead of going through the `deposit` function and sway the results.

- Future integration with LSTs with less than 1e18 decimals: Our team uncovered a vulnerability based on the developer's assumption that all LST tokens used as supported assets for the system will use 1e18 decimals. The developers communicated in a private discord thread that they are planning to add more supported LST assets in the future if EigenLayer expands. 
```
Gus — Yesterday at 6:38 PM
When Eigenlayer adds/accepts more LSTs, we will want to support those as well
```
If in the future additional assets are supported with a decimal of more/less than 1e18, the calculaction used in the for-loop at `LRTOracle::getRSETHPrice#L66` will return heavily skewed results and thus minting a way bigger/smaller amount than it should.

**5.2 Centralization Risks**

Parts of the functionality of the system is locked behind priviliged-role access control modifiers, functions such as adding new supported assets, setting their deposit limits, increasing the node delegator count, pause/unpause of the protocol, transfer of funds between contracts and EigenLayer strategies and oracle integration.

All of these are expected, except maybe letting users control how many of their funds, and when, are allocated to EigenLayer strategies. The codebase is still in a very early stage (especially seeing a `Withdraw` function is missing), and it might be in the plans of the developers to allow for such actions to be decided by the users, but as it stands right now, I feel like that part is rather centralized and an improvement to this would be to allow the users to choose.



***


# Disclosures

C4 is an open organization governed by participants in the community.

C4 Audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
