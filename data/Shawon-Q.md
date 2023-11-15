# Use a hardcoded address instead of address(this)

It can be more gas-efficient to use a hardcoded address instead of the address(this) expression, 
especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract’s address from its bytecode,
 which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address,
 you can avoid this additional operation and reduce the overall gas cost of your contract.

``` solidity
file: src/NodeDelegator.sol

63 uint256 balance = token.balanceOf(address(this));

103 IEigenStrategyManager(eigenlayerStrategyManagerAddress).getDeposits(address(this));

111 assetBalances[i] = IStrategy(strategies[i]).userUnderlyingView(address(this));
 
123 return IStrategy(strategy).userUnderlyingView(address(this));

```

 # If already paused/unpaused, revert.

``` solidity
file: utils/LRTDeposit.sol 

 207    /// @dev Triggers stopped state. Contract must not be paused.
 208   function pause() external onlyLRTManager {
 209       _pause();
 210   }

 212  /// @dev Returns to normal state. Contract must be paused
 213   function unpause() external onlyLRTAdmin {
 214      _unpause();
 215  }

```
the dev comments states that the paused function must be called when it is not already paused, Same with the unpaused function.
But we see no such checks inside those functions to check if the contracted has already been paused by another LRTManager before
executing the function.

Recommending to add :

``` solidity
function pause() external onlyLRTManager {
 if (paused=true){
revert };
... 

```

``` solidity
 function unpause() external onlyLRTAdmin {
 if (unpaused=true){
revert };
...

```

Instances: `NodeDelegator.sol` , `LRTOracle.sol` , `RSETH.sols`