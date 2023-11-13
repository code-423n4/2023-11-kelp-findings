### Contextualizing findings

I found that the code did not compile and eventually realised it was an OpenZeppelin library which was wrongly speicified which is the source of one of my findings.

The well known "first depositor share inflation attack" for ERC4626 vaults seems like it also applies here even though it is a staking protocol and not an ERC4626 vault

Another issue considers rounding/precision loss. Rounding should always be against the user, so attackers cannot mint small amounts and get more than they should due to precision loss in their favor. However there is an instance of rounding in depositors favor which can be abused.

### Approach Taken in Evaluating the Codebase

Normally, experimenting with different input parameters and external functions that are not admin restricted is a good way to find bugs. However almost everything is restricted in Kelp. Therefore a different approach is needed.

Instead, I focused on:

- Does the code actually do what its supposed to?
- Does it integrate properly with Chainlink and EigenLayer?
- Can the maths, especially rounding, have errors?
- First deposit inflation attacks?
- Correct use of decimals

So basically code mistakes rather than attack vectors.

### Codebase Quality Analysis

The most pertinent feature of the codebase is that almost everything is restricted by access controls to privileged roles. This makes it quite difficult for an attacker to come up with creative attack vectors, as you can just call the `depositAsset` function with a limited parameter set. 

The codebase complexity was low (a good thing). Functions are very short and it is clear what they do. There was no complex maths or interactions.

One strange thing about the codebase is that ReEntrancy Guard is used for Admin Functions. Perhaps it would reduce some rug attack vectors, but admin is a trusted role, so it may not be necessary.

One noticable code smell that could lead to vulnerabilities is the use of `balanceOf`. This is easy to manipulate by sending tokens to the contract directly, which can break the internal accounting.

### Centralization Risks

The codebase is highly centralized. As noted in the quality analysis, almost everything is access controlled by manager. The manager can add nodes, change maximum nodes, pause, unpause, direct funds among other actions. However, this is by design for the protocol and admin is assumed to be a trusted role for the audit.

### Mechanism Review

Users deposit assets through the `depositAssets` function.

They get staking tokens proportional to the assets they deposit.

The asset value and ratios are determined by `LRTOracle.sol`. Currently the plan is to call `ChainlinkPriceOracle.sol` to get the chainlink price feed of assets compared to ETH. It is important to note that with `/ETH` pairs, Chainlink uses `18 decimals` for the return value of the oracle price.

The assets/staking is forwarded to EigenLayer.

Admin can perform pause/unpause functionality as well as updating nodeOperators and parameters.

### Systemic Risks

Since the admin controls so many important functions, the systemic risks are mainly related to private keys being lost and admin abusing power.

There are risks with EigenLayer. Since there are many different staking tokens, the depegging of a single token can put the whole EigenLayer system at risk. This also propogates to KelpDao.







### Time spent:
15 hours