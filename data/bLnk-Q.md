# Storage Variable used in Abstract Contract
## Summary
Contract RSETH is upgradeable and it is inheriting from LRTConfigRoleChecker and if in the future an additional variable is to be added it can create an issue because there is no storage gap or namespaced storage.

## Details
If new variables are added in the future to both RSETH and LRTConfigRoleChecker then whatever value RSETH variables have will be lost as they will as LRTConfigRoleChecker vars are sharing storage slots with them.

## Impact
Data loss in some of the new variables that occupy the same storage slot

## PoC
    abstract contract LRTConfigRoleChecker {
      ILRTConfig public lrtConfig; //slot1
      uint256 public lrtVar; //slot2
      ...
      }

and

    contract RSETH is ... {
      bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE"); //slot1
      bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE"); //slot1
      uint256 public rsethVar; //slot2
      ...
      }

When upgraded rsethVar will be overwritten by lrtVar

## Recommendation
Use structs for storage variables with a variable for storage location for it. keccak256 can be used to calculate the storage location. For reference check out: 
https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC20/ERC20Upgradeable.sol

Alternatively an array can be added in LRTConfigRoleChecker to create a storageslot gap.