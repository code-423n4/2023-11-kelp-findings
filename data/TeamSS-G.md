## Disclaimer 
*Gas findings in this report has been thoroughly tested to hold no inherent risk (security or otherwise) to the operations of the protocol but we strongly advise the Kelp DAO team to properly test before applying the recommended fix.*

## [G-0] Use modifier instead of a function for zero address checks. 
In multiple instances of the Kelp DAO codebase, `UtilLib::checkNonZeroAddress` is used to check for the presence of _zero address_ in addresses passed as arguments to core protocol functions. The problem with this approach is that it consumes more gas vs a modifier version. Although, it is worth noting that modifiers can't be defined in Libraries. For this, we suggest the usage of a helper/wrapper contract to contain the zero address modifier logic. It is true that modifiers 

[Reference](https://github.com/code-423n4/2022-01-elasticswap-findings/issues/77)

#### Instances:  used across 6 contracts. 
example: 
```
    function updatePriceFeedFor(address asset, address priceFeed) external onlyLRTManager onlySupportedAsset(asset) {
        UtilLib.checkNonZeroAddress(priceFeed); // This line. 
        assetPriceFeed[asset] = priceFeed;
        emit AssetPriceFeedUpdate(asset, priceFeed);
    }
```

#### POC

```
// SPDX-License-Identifier: MIT
 
 pragma solidity 0.8.21; 


/// @notice helper contract with the modifier for zero address check.
 contract Modifier {
   // deployment tx cost: 66854 gas
   // usage cost:  685 gas 

    error ZeroAddressNotAllowed();

   modifier checkNonZeroAddress(address address_) {
        if (address_ == address(0)) revert ZeroAddressNotAllowed();
        _;
    }

    
 }

 /// @notice Library with the function with zero address check. 

 library Function {
   // deployment tx cost: 72062 gas
   //  731 gas 
    error ZeroAddressNotAllowed();

    function checkNonZeroAddress(address address_) internal pure {
        if (address_ == address(0)) revert ZeroAddressNotAllowed();
    }

     
 }

 /// @dev contract to test the usage of both approcahes. 

 contract KelpGasCost is Modifier  {

   function updatePriceFeedFor0(address asset, address /*priceFeed*/) external pure  {
      Function.checkNonZeroAddress(asset);
      
      // do stuff. 
    }

     function updatePriceFeedFor1(address asset, address /*priceFeed */) external pure checkNonZeroAddress(asset)  {
      // do stuff. 
      
    }
 }
 ```


#### Recommmendation
The modifier approach is more gas efficient. thus, we recommend it as they can be inherited and overridden by derived contracts as needed. 

