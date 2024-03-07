Precise Cedar Carp

medium

# NO check for blacklist contract in bridge contract

## Summary

Blcaklist.sol is made in telcoin but only checks have been put in stablecoin.sol.

## Vulnerability Detail

There is no check for Blacklist user by protocol in Bridge Realy.sol

## Impact

Balcklisted user can transfer tokens through in bridge contract

## Code Snippet
https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/bridge/BridgeRelay.sol#L67

## Tool used



Manual Review

## Recommendation

check shoul be there ike stable coim
