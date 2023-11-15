# Analysis

## Table of Contents

- [Approach](#approach)
- [Architecture Overview](#architecture-overview)
- [Testability](#testability)
- [Architectural Risks](#architectural-risks)
- [Summary of Reported Findings](#summary-of-reported-qa-findings)
- [Learnt About](#learnt-about)
- [Other Recommendations](#other-recommendations)
- [Security Researcher Logistics](#security-researcher-logistics)
- [Conclusion](#conclusion)

## Approach

The auditing process began even before the contest started, grasping a high level but brief overview of the entire EigenLayer ecosystem and how KELP plans on integrating with this, i.e glancing through both [the written blogs](https://blog.kelpdao.xyz/) & the [official repo of the EigenLayer contracts](https://github.com/Layr-Labs/eigenlayer-contracts/blob/master/src/contracts/strategies/StrategyBaseTVLLimits.sol).

Once the contest began, the first few hours was dedicated to examining the smart contracts in scope, after this review, a few hours was spent on researching some of the supported assets, which was then followed by a meticulous, line-by-line security review of the sLOC in scope, beginning from the cotracts small in size like `Utils.sol` ascending to the bigger contracts like the`LRTConfig.sol`. The concluding phase of this process entailed interactive sessions with the developers on Discord. This fostered an understanding of unique implementations and facilitated discussions about potential vulnerabilities and observations that I deemed significant, requiring validation from the team.

## Architecture Overview

### [UtilLib](https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/UtilLib.sol)

- **Purpose**: This utility library provides essential helper functions that are used across various contracts. I
- **SLOC**: 7

### [LRTConstants](https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConstants.sol)

- **Purpose**: Acting as a repository of constant variables, this contract ensures consistency and uniformity across the protocol.
- **SLOC**: 10

### [ChainlinkPriceOracle](https://github.com/code-423n4/2023-11-kelp/blob/main/src/oracles/ChainlinkPriceOracle.sol)

- **Purpose**: This contract serves as an entry point to Chainlink oracles, enabling the protocol to query external price data.
- **SLOC**: 25

> This is a simple but interesting contract, so below attached is a graphic representation.

![](https://user-images.githubusercontent.com/107410002/282992868-449d3698-4b15-42cb-abd0-2d9b4e52a3b3.png)

### [LRTConfigRoleChecker](https://github.com/code-423n4/2023-11-kelp/blob/main/src/utils/LRTConfigRoleChecker.sol)

- **Purpose**: This contract plays a key role of implementing diligent checks and manages the roles assigned to different participants in the Kelp protocol, ensuring that functions are accessed by the right entities.
- **SLOC**: 33

### [RSETH](https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol)

- **Purpose**: This contract is just the receipt token for users depositing this ERC20 token contract is a representation of users' stakes in the project.
- **SLOC**: 45

### [LRTOracle](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTOracle.sol)

- **Purpose**: As an oracle contract, LRTOracle is crucial for fetching real-time price data of supported LSTs (Liquid Staking Tokens).
- **SLOC**: 60
  > This contract plays a very important role too, below attached is a graphic representation of the contract.

![](https://user-images.githubusercontent.com/107410002/282994040-82f06680-f6c0-4892-b4d1-257c6b90ce58.png)

### [NodeDelegator](https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol)

- **Purpose**: This contract acts as a financial conduit within the Kelp ecosystem. It receives funds from the LRTDepositPool and responsibly delegates them to EigenLayer strategies. By managing these funds, the NodeDelegator plays a pivotal role in liquidity provision and in the broader strategic financial management of the Kelp project.
- **SLOC**: 65

![](https://user-images.githubusercontent.com/107410002/282993034-22e83e42-5709-466a-847f-a12828fe4e94.png)

### [LRTDepositPool](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol)

- **Purpose**: Serving as the gateway for users' investments, this contract is the first point of contact for funds entering Kelp. It manages the users' deposits, linking them with the NodeDelegators for further action.
- **SLOC**: 97

![](https://user-images.githubusercontent.com/107410002/282993164-d245e7ee-f8ae-4b9b-876b-6babbc9b8ca5.png)

### [LRTConfig](https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTConfig.sol)

- **Purpose**: This contract is responsible for storing and managing the configuration of the entire product. From holding addresses of other critical contracts to maintaining overarching settings, the LRTConfig ensures that the Kelp project remains organized, coherent, and adaptable to changes.
- **SLOC**: 111

### Important Links

Check out the protocol's [Linktree.](https://linktr.ee/kelpdao)

## Testability

The system's testability is commendable being that is has a wide coverage, one thing to note would be the occasionally intricate lengthy tests that are a bit hard to follow, this shouldn't necessarily be considered a flaw though, cause once familiarized with the setup, devising tests and creating PoCs becomes a smoother endeavour. In that sense I'd commend the team for having provided a high-quality testing sandbox for implementing, testing, and expanding ideas.

## Architectural Risks

- Rebase LSTs that are integrated to the system like stETH, uses it's pure balance instead of using shares, which would be an issue whenever the balance/share ratio shifts, or better still protoccol can just implement the wrapped version of this token.
- Protocol does not take into account that prices returned from chainlink could be stale which could lead to the ingestion of wrong prices and potentially over/undervaluing the assets that's in the pool or the amount of rsETH to mint.
- Adding a non active asset, or providing the wrong oracle for an asset would lead to a massive over/under valuing of assets while minting.
- Additionally, generic pausing/unpausing responsibility of admins, could endanger protocol.
- Protocol in some cases currently uses the non-optimal oracle for assets, which also could lead to cases where assets are under/over valued, this is pertaining stETH whose USD feed is considered way safe than it's ETH counterpart.

## Summary of Reported Findings

During my security review, I identified several issues impacting the smart contract functionalities, each presenting unique challenges and potential risks to the system's overall integrity and compliance with multiple issues pertaining external protocol integration with KELP, particularly in price querying, which can disrupt staking and depositing, this is in relation to the current _Chainlink_ integration.

An attacker can grief honest users by front-running their deposit transactions in `LRTDepositPool.depositAsset()` with a minuscule deposit amount (as small as one wei), this eventually leads to user frustration and even protocol's inability to have an assurance that a  supported asset's limit would ever be reached.

Transfers in LRTDepositPool may fail with non-standard EIP20 tokens that revert instead of returning false on failure.

Risk of overvaluing or undervaluing assets in `rETH` minting due to Chainlink price feed's border prices, i.e when the prices of the supported assets goes higher/lower than the min/max circuit breakers.

Potential denial-of-service risk in depositing/staking rETH due to RocketPool's deposit delay feature if it ever gets turned on.

Tokens with proxy and implementation contracts (like cbETH/rETH) might change features, potentially breaking KELP's functionality.

QA finding on Lack of equality checks in setters like `updateAssetDepositLimit` can lead to unnecessary updates and inefficiencies.

Risk of denial-of-service in `depositAsset()` due to reliance on external Chainlink oracles, which could fail or be taken down.

Inaccurate valuation of stETH during rsETH minting could result in overvalued or undervalued minted rsETH, affecting fair asset exchange.

LRTOracle doesn't implement the `whenNotPaused()` modifier, unlike other contracts, potentially leading to operational inconsistencies, this seems to be a honest miss since the protocol inherits the `PausableUpgracdeable.sol` contract.

Suboptimal calculation of stETH's exchange rate using ETH feed, whereas the USD feed is safer, which may lead to incorrect valuations in the `totalETHInPool` and affect rsETH minting in `depositAsset()`.

Potential for minting incorrect `rsETH` amounts due to flawed asset price calculation from Chainlink oracles.

Redundant zero address checks in `LRTConfig::initialize()` lead to inefficiencies, as checks are already performed in `_setToken()`.

Potential inaccuracy of `rsETH` minting calculation which could lead to new depositors receiving a significantly better exchange rate.

## Learnt About

During the course of my security review, I had to research some stuffs to understand the context in which they are currently being used, some of which led to me learning new things and having refreshers on already acquainted topics. With this contest it was mostly LSTs, with them listed below:

- `cbETH`
- `rETH`
- Had a refresher on `stETH`

## Other Recommendations

### **Enhance Documentation Quality**

To start the docs are generally well-crafted. However, there are some gaps concerning what's within the scope. It's important to remember that code comments also count as documentation, and there have been some inconsistencies in this area. Comprehensive documentation not only provides clarity but also facilitates the onboarding process for new developers.

### **Onboard More Developers**

Having multiple eyes scrutinizing a protocol can be invaluable. More contributors can significantly reduce potential risks and oversights. Evidence of where this could come in handy can be gleaned from codebase typos. Drawing from the [broken windows theory](https://en.wikipedia.org/wiki/Broken_windows_theory), such lapses may signify underlying code imperfections.

### **Leverage Additional Auditing Tools**

Many security experts prefer using Visual Studio Code augmented with specific plugins. While the popular [Solidity Visual Developer](https://marketplace.visualstudio.com/items?itemName=tintinweb.solidity-visual-auditor) has already been integrated with the protocol, there's room for incorporating other beneficial tools.

### **Enhance Event Monitoring**

Current implementation subtly suggests that event tracking isn't maximized. Instances where events are missing or seem arbitrarily embedded have been observed. A shift towards a more purposeful utilization of events is recommended.

### **Improve Testability**

Whereas the previous comment on testing was commending it, this would however counter that and hint protocol to try hitting that 100% test mark, as with any good thing it could definitely get better.


### **Enhance Event Monitoring**

Current implementation subtly suggests that event tracking isn't maximized. Instances where events are missing or seem arbitrarily embedded have been observed. A shift towards a more purposeful utilization of events is recommended.

## Security Researcher Logistics

My attempt on reviewing the Codebase spanned around 25 hours distributed over 3/4 days:

- 1 hour dedicated to writing this analysis.
- 3 hours researching on implementations novel to me (i.e LSTs), being that this is my first Code4rena contest that has them at the core of the protocol.
- 2 hours over the course of the 3 days _(~40 minutes per day)_ was spent in the discord chat to get more ideas based on questions other security researchers ask and explanation provided by protocol developers in order to understand the protocol better.
- The remaining time was spent on finding issues and writing the report for each of them on the spot, later on _editing a few with more knowledge gained on the protocol as a whole, or downgrading them to QA reports._

## Conclusion

The codebase was a very great learning experience, though it was a pretty hard nut to crack, being that it's like my first contest that deals with LSTs, additionally it has a little amount of nSLOC which as usual _the lesser the lines of code the lesser the bug ideas_ , nonetheless it was a great ride seeing the unique implementations employed in the codebase.




### Time spent:
25 hours