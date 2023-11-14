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