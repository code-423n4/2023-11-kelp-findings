## [L-01] _safeMint() should be used rather than _mint() wherever possible
_mint() is [discouraged](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L271) in favor of _safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver. Both open [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L238-L250) and [solmate](https://github.com/Rari-Capital/solmate/blob/4eaf6b68202e36f67cab379768ac6be304c8ebde/src/tokens/ERC721.sol#L180) have versions of this function so that NFTs aren’t lost if they’re minted to contracts that cannot transfer them back out.


```solidity
File:   src/RSETH.sol
48     _mint(to, amount);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L48


## [L-02] Use `safetransfer` Instead Of `transfer`
It is good to add a require() statement that checks the return value of token transfers or to use something like OpenZeppelin’s safeTransfer/safeTransferFrom unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.
For example, Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)'s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert.

```solidity
File:  src/LRTDepositPool.sol
194   if (!IERC20(asset).transfer(nodeDelegator, amount)) {
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/LRTDepositPool.sol#L194

```solidity
File:  src/NodeDelegator.sol
86  if (!IERC20(asset).transfer(lrtDepositPool, amount)) {
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/NodeDelegator.sol#L86


Recommended Mitigation Steps
Consider using safeTransfer/safeTransferFrom or require() consistently.


## [L-03] no check if the burn amount is zero or not
if the amount is zero so the unhecked block will divided zero on 3 and we use gas for nothing ! if we set zero we may pass the _burn checks, i know it is passed by only permit but it's better to avoid this happen because it's seting by human and it means it can be set with 0 balance to burn !

```solidity
File:   src/RSETH.sol
55    _burn(account, amount);
```
https://github.com/code-423n4/2023-11-kelp/blob/main/src/RSETH.sol#L55
