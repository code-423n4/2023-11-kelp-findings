## Overview of Kelp platform
Kelp is a platform for minting re-staking erc20 token `rsETH` for stETH, rETH and cbETH, and it is based on Eigenlayer protocol.

It is still in early age of Kelp platform, this is way the audit is just with 504 Sloc. 90% percent of the function are not fully finished and a lot of them are centralized and can be used only by the manager or the admin. The comments are an effective way to understand the codebase and they did a good job writing them. 

In this audit of Kelp there are 5 core contracts – 
Config, DepositPool, NodeDelegator, Oracle and the RSETH ERC-20 token.

In the Config contract, all of the function can be used only by the Manager of the platform or the Admin. And they are used to set or get things like: supported Assets, Deposit limit, or setting tokens.

In the DepositPool contract, the only function that can be used by users is DepositAsset, which is the only function that users can use in the whole protocol for that moment, expect the view functions.
The nodeDelegator contract, is used to transfer assets to the strategy. Or transfer them back to the DepositPool

The Oracle contract, get the price for the assets and for RSETH with the help of chainlink Oracle.

The RSETH contract, is the ERC-20, which is the re-staking token  

## My Thoughts
From mine perspective, the Kelp protocol is still in incredibly young age to be said anything for it. In the future it needs to be more efficient, for the reason that it cannot do anything at the moment.

## Audit approach
1.	Read the audit Readme.md and took the required notes.
Erc20
Re-staking token
Eigenlayer
The test coverage is 98%
Pausable function

2.	Read their blog from -> https://linktr.ee/kelpdao
3.	Then I setup my testing environment things. Run the tests to checks all test passed.
4.	Finally, I started with auditing the codebase in-depth. I started understanding the code line by line and took the necessary notes to ask some questions to sponsors.

## Stages of audit 

The first stage of the audit: Reading and understanding the documentation of Kelp protocol and the Eigenlayer.
This phase of the audit aimed to ensure the change from Eigenlayer.

The second stage of the audit: In the second stage of the audit for the Kept platform, the focus shifted towards understanding the protocol usage in more detail. This involved identifying and analyzing the important contracts and functions within the system. By examining these key components of the audit, I aimed to gain a comprehensive understanding of the protocol’s functionality and potential risks.

The third stage of the audit: thinking how malicious users can harm the protocol or the others. Doubting every calculation and vulnerable areas within the protocol. 

## Possible Systemic Risks

Centralization Risk: All of the function are centralized expect 1, everything can go wrong, because of the admins, if they set something wrong

Eigenlayer risks: creating platform based on another can be extremely risky, because if Eigenlayer change something dramatically this will open the door for the malicious users.

## Architecture Recommendations

I can’t possibly have any recommendations for this platform. It is written in the best way, with all of the comments you need. The test are tremendously structure in order. Everything is so simple and readable.

## Codebase quality 
The overall quality of the codebase for PoolTogether can be classified as ”Good“. Expect the documentation, it should be re-written.





### Time spent:
6 hours