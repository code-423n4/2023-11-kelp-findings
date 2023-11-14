## rounding down error when depositing stETH tokens
The `transfer` function in stETH token first converts the amount that is being transferred to  correct amount of shares using getSharesByPooledEth function:
```
    function getSharesByPooledEth(uint256 _ethAmount) public view returns (uint256) {
        return _ethAmount
            .mul(_getTotalShares())
            .div(_getTotalPooledEther());
    }
```
this conversion causes a rounding down error. for example, if a user deposited 1 ether of stETH into the pool, (1 ether - 1) will be received by the contract, this means both the emitted event (AssetDeposit) and calculation of rsETH to mint, are done using a wrong number.

POC:
Tests were done on ETH mainnet (forked):
```
    function test_DepositSTETHFlooring() external {
        uint256 amountToDeposit = 1 ether;
        vm.startPrank(manager);
        lrtConfig.updateAssetDepositLimit(address(stETH), amountToDeposit);
        vm.stopPrank();

        vm.startPrank(alice);
        //approve the pool
        stETH.approve(address(lrtDepositPool), ~uint256(0));

        //wrong even emitted, contract received 1 ether - 1, not 1 ether
        vm.expectEmit(true, true, true, false);
        emit AssetDeposit(address(stETH), 1 ether, rseth.balanceOf(alice));
        lrtDepositPool.depositAsset(address(stETH), amountToDeposit);

        //stETH balance of contract is  1 ether - 1
        assertEq(stETH.balanceOf(address(lrtDepositPool)), 1 ether - 1);

        vm.stopPrank();
    }
```
Mitigation:
Consider calculating how much asset deposit pool actually received instead of relaying on `depositAmount`



## Not using SafeERC20 when checking return value of an ERC20 transfer:
Impact:
some sections of the code involve direct checks of the return value from ERC20 `transfer` and `transferFrom` functions. This approach does not utilize the SafeERC20 library, which can lead to potential issues with certain ERC20 tokens that do not return a boolean (true) if their transfers were successful:

NodeDelegator : when transferring tokens back to deposit pool using `transferBackToLRTDepositPool`
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L86-L88
LRTDepositPool : when transferring tokens to a node delegator using `transferAssetToNodeDelegator`
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L194-L196
LRTDepositPool : when transferring tokens from user to the deposit pool using `depositAsset`
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L136-L138

Some older ERC20 tokens, such as USDT, do not return a boolean value to indicate the success or failure of transfer or transferFrom operations. Consequently, the return value for these tokens will always be false, regardless of the actual outcome of the transfer. While the default assets of the system (e.g., cbETH, stETH, and rETH tokens) do return a true boolean upon successful transfers, reliance on direct return value checks poses compatibility risks with assets that may be integrated into the system in the future.
