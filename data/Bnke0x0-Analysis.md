
Kelp DAO eagerly embarks on the development journey of rsETH, a Liquid Restaked Token (LRT) meticulously crafted to enhance liquidity and flexibility within the DeFi landscape. This analysis delves into the intricate details of rsETH, shedding light on its underlying principles and technological foundations.

Fundamentally, an LRT serves as a fluid token, strategically supplying liquidity to assets that may otherwise lack market depth when staked in restocking platforms like EigenLayer. Notably, rsETH has successfully transitioned into the testnet phase, marking a significant milestone in its developmental trajectory. As we navigate through the intricacies of this project, let's explore a concise overview of the technological mechanisms propelling rsETH forward.



### Approach taken in evaluating the codebase
 

It is crucial to thoroughly review the recommended specifications and associated references before delving into the codebases line by line. I commenced by examining the generic contracts and systematically addressed each specific plugin. Immersing myself in the intricacies of the code logic proved both enjoyable and satisfying, especially when identifying potential vulnerabilities.


### Codebase quality analysis

- Kelp DAO exhibits a meticulously organized and modular design in its overall architecture, adhering to widely accepted governance patterns.

- The incorporation of the EigenLayer protocol.  introduces flexibility; 

- The codebase maintains a commendable level of quality, complemented by comprehensive comments that elucidate each step in the process. 

- Test quality is notably high, characterized by well-structured tests with an impressive coverage rate of 99%.

introducing fuzzing tests would serve to bolster test robustness even further.





### Centralization risks

Eliminating a singular vulnerability Centralization hazards associated with Kelp agreements


### Mechanism Review
1. LRTConfig:
The LRTConfig contract stands as a dynamic and upgradable component within the Kelp product infrastructure. Its primary functions encompass the storage of Kelp product configurations and the maintenance of crucial contract addresses associated with other elements within the Kelp ecosystem.

2. LRTDepositPool:
Operating as an upgradable contract, LRTDepositPool assumes the pivotal role of receiving user deposits within the Kelp product. This contract serves as a central hub from which deposited funds are subsequently channeled to NodeDelegator contracts.

3. NodeDelegator:
NodeDelegator contracts form a critical link in the financial flow within the Kelp product. Functioning as upgradable entities, these contracts receive funds from the LRTDepositPool and facilitate their delegation to the EigenLayer strategy. The allocated funds play a crucial role in providing liquidity within the EigenLayer protocol.

4. LRTOracle:
As another upgradable component, the LRTOracle contract assumes responsibility for retrieving the price data of Liquid Stacking Tokens from external Oracle services. This functionality is vital for maintaining accurate and up-to-date pricing information within the Kelp product ecosystem.

5. RSETH:
RSETH emerges as a receipt token integral to the deposit process within the Kelp product. Operating as an upgradable ERC20 token contract, RSETH serves as a testament to Stader Labs' commitment to flexibility and adaptability in its token-related functionalities.

In conducting this audit, the focus is placed on scrutinizing the security, robustness, and seamless integration of these contracts, thereby ensuring the overall stability and efficacy of the Kelp product. The careful examination of each contract's design and functionality will provide valuable insights into the resilience of Stader Labs' infrastructure and its readiness for deployment in real-world scenarios.

### Systemic risks

Following is my risk analysis of Kelp's main contracts, namely LRTConfig and LRTDepositPool, which contains the most logic.

### Time spent:
18 hours



### Time spent:
18 hours