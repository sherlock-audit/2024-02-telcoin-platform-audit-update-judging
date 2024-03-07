Radiant Tawny Dinosaur

medium

# No checks to prevent Blacklisted user from using protocol's function

## Summary
A blacklisted user of the protocol can call every function of smart contract as there are no checks to prevent blacklisted user from using protocol's function.

## Vulnerability Detail

-There is a blacklist contract which blacklists user from using protocol's function.
-But there are no checks in all the smart contracts which prevent blacklisted user from using the protocol's functions.
## Impact

-Blacklisted user would be able to call all the functions of the protocol even though the user is blacklisted which creates a major problem as protocol team has blacklisted the user because of malicious activities , now he can continue malicious activities even though the user is blacklisted. 

## Code Snippet
https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/bridge/BridgeRelay.sol#L44
https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/bridge/BridgeRelay.sol#L88
https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/stablecoin/Stablecoin.sol
## Tool used

Manual Review

## Recommendation
Check the user has been blacklisted or not by the protocol from `Blacklist.sol`