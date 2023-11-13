# Approach taken in evaluating the codebase

To understand protocol was used EigenLayer documentation.

Flow:
1) Start with understanding linked protocol. This protocol is EigenLayer. Read whitepaper and linked documentation first.
2) Then determine which part of EigenLayer protocol used by Kelp DAO. The audited protocol uses only Strategy Manager and Strategy. Current implementation of Kelp DAO doesn't support withdrawn.
3) Then proceed to manual code review.

# Architecture recommendations

Current architecture is well designed.

It's absolutely correct to separate DepositPool and NodeDelagator into two different contracts. Because, if one of these contract will be hacked, then protocol lost only one part.  

**But here is one detail, which is important to highlight.**
Withdraws. 
Current implementation doesn't support withdraws. 
Devs told that it will be added later. 
But, when we deposit some asset (like stETH and rETH) to Kelp -- we erase the boundaries of these tokens and after that moment it becomes unclear who contributed which tokens.
For example:
* Alice deposits 1 stETH, he got 1 rsETH;
* Bob  deposits 1 rETH, he got 1 rsETH;
* Now we have 2 rsETH and 1 stETH and 1 rETH, but don't know who initial depositor.

**Moreover, if you go to prod right now, it will be impossible to determine later who deposit which tokens.**

Need to support additional mapping of: `user => asset => initial asset amount`.

# Codebase quality analysis

Codebase is really good, all cases are handled correctly. 
Dev team writes good and clear comments and uses meaningful named-attributes in mapping.

However, devs uses transfer, transferFrom and approve instead of safe version of these functions. Better to use safe implementation by OppenZeppelin, because tokens, which will be added in future may return nothing in `transfer`.

*Additional notice:*
LRTConfig has `tokenMap` which is used for clarity description of what address responsible for. 
But current implementation gives possibility to update one of `isSupportedAsset` or `depositLimitByAsset` or `assetStrategy` keys without `tokenMap` modification. These should be fixed and work synchronously.

# Centralization risks

Current protocol's version is absolute centralized. Core contracts, and core functions: DepositPool, NodeDelegator is under modifiers. Only admin can move funds to strategy. 
After you enter to protocol, only admins can control your money. 
This behavior is not desired. 

Better develop or think about more freedom for users in terms of transfer funds.

# Mechanism review

The system consist of three main parts:

* First part is DepositPool with Oracle. 
This is main entry point. User deposits funds use DepositPool contract, and Oracle contract uses to calculate price;

* Second part is NodeDelegator part. This part responsible to communication with EigenLayer. Funds passed to this contract are passed to EigenLayer's Strategies, which bring profit;

* Last one part is RsETH token, when user enter to protocol via DepositPool he receives RsETH, which is proof of ownership.

So, whole protocol in a general sense, can be represented as the following scheme:
```

  User
   |
   | User's funds
   ∨
--------------- transfer assets  -----------------  enter into strategy --------------
| DepositPool | <============>   | NodeDelegator |  =====>              | EigenLayer |
---------------                  -----------------                      --------------
       ^
       | 
       | Request Price
       ∨
   ---------- 
   | Oracle |
   ----------
```

# Systemic risks

A systemic risk in this system is the dependence on ChaniLink.

In future, when protocol will have bigger liquidity, better to additionally fetch price from on chain dex, like it's implemented in GMX. 

Because, Oracle can be stopped, can be hacked, so instead of relay only on one system, better has fallback option. Or even better to use two price providers (Oracle and DEX) and use mean price.



### Time spent:
12 hours