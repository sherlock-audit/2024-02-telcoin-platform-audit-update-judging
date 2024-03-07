Hollow Cotton Ape

medium

# Missing blacklist check beforeTokenTransfer allows anyone to bypass the blacklist mechanism

## Summary
See Detail.
## Vulnerability Detail
A user can be blacklisted, restricting any stablecoin transfer from that address. Currently, due to the absence of a check before the token transfer, a blacklisted address can transfer from/to without restriction. 

## Impact
Bypassing the blacklist mechanism.

## Code Snippet
https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/stablecoin/Stablecoin.sol
https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/util/abstract/Blacklist.sol#L62
## Tool used

Manual Review

## Recommendation
In Stablecoin.sol, override _beforeTokenTransfer to add the following check; 
```solidity
require(!blacklisted(destination), "Stablecoin: destination cannot be blacklisted address");
```