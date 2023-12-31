# QA Report - Kelp DAO Audit Contest | 10 Nov 2023 - 15 Nov 2023

---

# Executive summary

### Overview

|               |                                                                          |
| :-----------  | :----------------------------------------------------------------------- |
| Project Name  | Kelp DAO                                                                 |
| Repository    | https://github.com/code-423n4/2023-11-kelp                               |
| Website       | [Link](https://linktr.ee/kelpdao)                                        |
| Twitter       | [Link](https://twitter.com/KelpDAO)                                      |
| Methods       | Manual Review                                                            |
| Total SLOC    | 504 over 9 contracts                                                     |

### Findings Summary

| ID                | Title |  Severity                  |
| ----------------- | ----- |  ------------------------- |
| [L&#x2011;1](#l1-do-not-allow-the-modification-of-the-asset-for-particular-eigenlayer-strategy-after-amount-of-the-specific-asset-has-already-been-deposited-into-the-eigenlayer-strategy)     | Do not allow the modification of the `asset` for particular `EigenLayer Strategy` after `amount` of the specific `asset` has already been deposited into the `EigenLayer Strategy` | _Low_ |
| [L&#x2011;2](#l2-a-malicious-lrt-manager-can-disrupt-the-deposit-process-for-a-particular-asset-by-setting-depositlimitbyassetasset-to-a-value-smaller-than-the-actual-total-supply-of-this-asset-within-contracts-consequently-the-depositing-function-will-consistently-revert-due-to-underflow)     | A malicious `LRT Manager` can disrupt the deposit process for a particular asset by setting `depositLimitByAsset[asset]` to a value smaller than the `actual total supply of this asset within contracts`. Consequently, the depositing function will consistently revert due to underflow | _Low_ |
| [L&#x2011;3](#l3-ownership-can-be-renounced)     | Ownership Can Be Renounced | _Low_ |
| [L&#x2011;4](#l4-possible-dos-and-out-of-gas-errors)     | Possible DoS and Out of Gas errors | _Low_ |
| [NC&#x2011;1](#nc1-implement-a-check-to-determine-if-the-nodedelegator-already-exists-within-the-lrtdepositpoolsoladdnodedelegatorcontracttoqueue-function)   | Implement a check to determine if the `NodeDelegator` already exists within the `LRTDepositPool.sol#addNodeDelegatorContractToQueue()` function | _Non Critical_ |
| [NC&#x2011;2](#nc2-a-malicious-user-can-disrupt-the-logic-of-the-lrtdepositpoolsol-contract-by-directly-transferring-assets-to-the-lrtdepositpool-nodedelegator-or-eigenlayer-strategy-contracts)   | A malicious user can disrupt the logic of the `LRTDepositPool.sol` contract by directly transferring `assets` to the `LRTDepositPool`, `NodeDelegator` or `EigenLayer Strategy` contracts | _Non Critical_ |
| [NC&#x2011;3](#nc3-grant-the-lrt-manager-permissions-during-the-initialization-of-the-lrtdepositpool-contract)   | Grant the `LRT Manager` permissions during the initialization of the `LRTDepositPool` contract | _Non Critical_ |
| [NC&#x2011;4](#nc4-grant-minter_role-and-burner_role-permissions-during-the-initialization-of-the-rseth-contract)   | Grant `MINTER_ROLE` and `BURNER_ROLE` permissions during the initialization of the `RSETH` contract | _Non Critical_ |
| [NC&#x2011;5](#nc5-implement-a-function-for-removing-assets-from-supportedassets-only-if-the-asset-has-not-already-been-transferred-to-the-nodedelegator-contract)   | Implement a function for removing assets from `supportedAssets` only if the asset has not already been transferred to the `NodeDelegator` contract | _Non Critical_ |

##

----

##

## [L&#x2011;1] Do not allow the modification of the `asset` for particular `EigenLayer Strategy` after `amount` of the specific `asset` has already been deposited into the `EigenLayer Strategy`

### Explanation

The `LRTConfig` contract has a vulnerability that allows the modification of the `asset` for a particular `EigenLayer Strategy` after an `amount` of the specific `asset` has already been deposited into the `EigenLayer Strategy`. This can potentially disrupt the logic of the system, leading to unintended consequences and security risks like permanently locked users funds.

The vulnerability arises from the ability to change the `asset` for a particular `EigenLayer Strategy` even after an `amount` of the specific `asset` has already been deposited into the `EigenLayer Strategy`. The sequence of events leading to this issue is as follows:

1. An `EigenLayer Strategy` is configured with a specific `asset`.

2. An `amount` of that `asset` is deposited into the `EigenLayer Strategy`.

3. The `LRTConfig` contract allows the `asset` for the same `EigenLayer Strategy` to be modified, potentially switching to a different asset.

4. This modification can disrupt the logic and expectations of the system, as it may not be designed to handle asset changes for an active `EigenLayer Strategy`.

### Impact

Changing the `asset` for an active `EigenLayer Strategy` could result in the permanently lock/loss of funds or assets being incorrectly handled.

### Recommendation

Implement a restriction that prevents the modification of the `asset` for a particular `EigenLayer Strategy` after any `amount` of the specific `asset` has been deposited into that Strategy. This can be achieved by adding a check in the `setAsset()` function to disallow changes when there is a non-zero balance of the `asset` in the `EigenLayer Strategy`.

---

## [L&#x2011;2] A malicious `LRT Manager` can disrupt the deposit process for a particular asset by setting `depositLimitByAsset[asset]` to a value smaller than the `actual total supply of this asset within contracts`. Consequently, the depositing function will consistently revert due to underflow

### Explanation

The `LRTConfig` contract has a vulnerability that allows a malicious `LRT Manager` to disrupt the deposit process for a particular asset by setting `depositLimitByAsset[asset]` to a value smaller than the `actual total supply of this asset within the contracts` (`LRTDepositPool`, `NodeDelegator` and `EigenLayer Strategy` contracts). This vulnerability can cause the depositing function to consistently revert due to underflow, effectively preventing legitimate deposits.

The vulnerability occurs due to the lack of proper checks and balances in the `LRTConfig` contract. The sequence leading to this issue is as follows:

1. The `LRT Manager` has the authority to set the `depositLimitByAsset` for a particular asset.

2. The `LRT Manager` sets `depositLimitByAsset[asset]` to a value smaller than the actual total supply of that asset within the contract.

3. Users attempt to deposit the asset into the contract.

4. Since the `depositLimitByAsset` is smaller than the total supply, the depositing function consistently reverts due to an underflow error when calculating available deposit limits.

5. Legitimate deposits are prevented, causing a disruption in the deposit process for the affected asset.

### Impact

The impact of this vulnerability is severe, as it can disrupt the deposit process for a particular asset. Users attempting to deposit the asset will encounter consistent reverts, making it impossible to deposit funds as intended. This can lead to a breakdown in the functionality of the ecosystem.

Likelihood: Low

### Recommendation

To mitigate this vulnerability, it is crucial to implement proper checks when setting the `depositLimitByAsset` to ensure that it cannot be set to a value smaller than the `actual total supply of the asset within the contracts` (`LRTDepositPool`, `NodeDelegator` and `EigenLayer Strategy` contracts).

---

## [L&#x2011;3] Ownership Can Be Renounced

### Explanation

If the owner renounces their ownership in `LRTConfig.sol`, all ownable contracts will be left without an owner. As a result, any function guarded by the `onlyOwner` modifier will no longer be executable. Additionally, the` DEFAULT_ADMIN_ROLE` can be renounced or revoked.

### Recommendation

Confirm whether this is the intended behavior. If not, override and disable the `renounceOwnership()` and `revokeRole()` functions in the affected contracts. For added security, consider implementing a two-step process when transferring ownership of the contract (e.g., using Ownable2Step from OpenZeppelin).

---

## [L&#x2011;4] Possible DoS and Out of Gas errors

### Description

The `LRTDepositPool` contract contains `nodeDelegatorQueue` array, which can lead to uncontrolled growth of the array over time. This uncontrolled growth can result in increased gas costs and potential out-of-gas errors during function calls that read or modify the array. Additionally, it may expose the contract to a denial-of-service (DoS) by causing transactions to run out of gas.

### Impact

- Increased gas costs: As the `nodeDelegatorQueue` array grows, the gas required to execute functions that modify the array (such as `addNodeDelegatorContractToQueue` and `transferAssetToNodeDelegator`) also increases, potentially leading to higher transaction fees for users.

- Possible out-of-gas errors: If the `nodeDelegatorQueue` array becomes too large, it may exceed the gas limit for Ethereum transactions, resulting in out-of-gas errors and failed transactions.

- Denial-of-Service (DoS) potential: An attacker could exploit the uncontrolled growth of the `nodeDelegatorQueue` to launch a DoS attack by repeatedly calling functions that modify the array, causing legitimate transactions to fail due to high gas costs.

### Explanation

The vulnerability arises from the fact that the `nodeDelegatorQueue` array is never reduced in size by removing elements. The contract allows NodeDelegator contract addresses to be added to the queue via the `addNodeDelegatorContractToQueue` function but does not provide a mechanism to remove addresses. As a result, the array can grow indefinitely.

```js
function addNodeDelegatorContractToQueue(
    address[] calldata nodeDelegatorContracts
) external onlyLRTAdmin {
    uint256 length = nodeDelegatorContracts.length;
    if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
        revert MaximumNodeDelegatorCountReached();
    }

    for (uint256 i; i < length; ) {
        UtilLib.checkNonZeroAddress(nodeDelegatorContracts[i]);
        nodeDelegatorQueue.push(nodeDelegatorContracts[i]);
        emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]);
        unchecked {
            ++i;
        }
    }
}
```

The above function allows `NodeDelegator` contract addresses to be added to the `nodeDelegatorQueue`, but there is no corresponding function to remove addresses from the queue when they are no longer needed or have fulfilled their purpose.

### Proof of Concept

This vulnerability can be demonstrated by deploying the `LRTDepositPool` contract and repeatedly adding NodeDelegator contract addresses to the `nodeDelegatorQueue` using the `addNodeDelegatorContractToQueue` function. As addresses are added, the array will grow without limit.

```js
// Deploy LRTDepositPool contract
// ...

// Repeatedly add NodeDelegator contract addresses to the queue
for (uint256 i = 0; i < 1000; i++) {
    lrtDepositPool.addNodeDelegatorContractToQueue([nodeDelegatorAddress]);
}
```

As a result, the `nodeDelegatorQueue` will continue to grow, potentially leading to increased gas costs and potential out-of-gas errors during the addition of addresses.

### Tools Used

Manual code review and analysis.

### Recommended Mitigation Steps

To address this vulnerability, it is recommended to implement a mechanism to remove or "pop" elements from the `nodeDelegatorQueue` when `NodeDelegator` contract addresses are no longer needed or have fulfilled their purpose. This can be achieved by adding a function to remove addresses from the queue or `pop` elements in the end of `transferAssetToNodeDelegator()` function when the asset is already transferred (the second possible solution is just an example mitigation step).

---

---
---

## [NC&#x2011;1] Implement a check to determine if the `NodeDelegator` already exists within the `LRTDepositPool.sol#addNodeDelegatorContractToQueue()` function

### Explanation

In the `LRTDepositPool.sol` contract, specifically within the `addNodeDelegatorContractToQueue()` function, there is currently no check to determine if a `NodeDelegator` contract address already exists in the `nodeDelegatorQueue` before adding it. This omission could lead to the unintended addition of duplicate addresses to the queue, potentially causing issues when processing deposits or transfers to these contracts.

### Recommendation:

It is advisable to implement a check within the `addNodeDelegatorContractToQueue` function to ensure that duplicate `NodeDelegator` addresses are not added to the `nodeDelegatorQueue`.

Here is an example of how the function could be modified to include the check:

```diff
    function addNodeDelegatorContractToQueue(
        address[] calldata nodeDelegatorContracts
    ) external onlyLRTAdmin {
        uint256 length = nodeDelegatorContracts.length;
        if (nodeDelegatorQueue.length + length > maxNodeDelegatorCount) {
            revert MaximumNodeDelegatorCountReached();
        }

        for (uint256 i; i < length; ) {
+           if (nodeDelegatorQueue[nodeDelegatorContracts[i]] != address(0)) (
+               revert();
+           )

            UtilLib.checkNonZeroAddress(nodeDelegatorContracts[i]);
            nodeDelegatorQueue.push(nodeDelegatorContracts[i]);
            emit NodeDelegatorAddedinQueue(nodeDelegatorContracts[i]);
            unchecked {
                ++i;
            }
        }
    }
```

By implementing this check, the `addNodeDelegatorContractToQueue` function will prevent the addition of duplicate `NodeDelegator` contract addresses to the `nodeDelegatorQueue`, ensuring the integrity of the queue and avoiding potential issues that may arise from duplicates.

---

## [NC&#x2011;2] A malicious user can disrupt the logic of the `LRTDepositPool.sol` contract by directly transferring `assets` to the `LRTDepositPool`, `NodeDelegator` or `EigenLayer Strategy` contracts

### Explanation

This will disrupt the intended `deposit()` functioning of the contract and the associated ecosystem, as the contract relies on controlled depositing procedures to ensure proper asset management.

---

## [NC&#x2011;3] Grant the `LRT Manager` permissions during the initialization of the `LRTDepositPool` contract

---

## [NC&#x2011;4] Grant `MINTER_ROLE` and `BURNER_ROLE` permissions during the initialization of the `RSETH` contract

---

## [NC&#x2011;5] Implement a function for removing assets from `supportedAssets` only if the asset has not already been transferred to the `NodeDelegator` contract

### Explanation

This will prevent potential out of gas errors.

---
