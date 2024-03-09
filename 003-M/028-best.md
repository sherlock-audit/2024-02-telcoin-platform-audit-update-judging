Fierce Carbon Ant

medium

# Stablecoin blocklist feature is ineffective

## Summary
The blocklist feature on the stablecoin doesn't prevent users from sending/receiving tokens defeating the purpose of blocklisting.

## Vulnerability Detail
The [current](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/stablecoin/Stablecoin.sol) stablecoin contract is an upgrade of the contract from the [previous](https://github.com/sherlock-audit/2023-02-telcoin/blob/main/telcoin-audit/contracts/stablecoin/Stablecoin.sol) audit. They both implement a blocklist feature to prevent blocklisted users from transferring the tokens.

One of the major differences is the openzeppelin version in use, [5.0.1](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/21920190e0772afa18e7f856a036fea3ef5b9635/telcoin-contracts/package.json#L30) in the current codebase, [4.8.1](https://github.com/sherlock-audit/2023-02-telcoin/blob/97b2bcd78740a870159b25d2f04806068c1c4c1f/telcoin-audit/package.json#L28) in the previous. This is especially important as the inherited ERC20upgradeable(inherited in ERC20PermitUpgradeable that the contracts inherit) handle the internal `_transfer` function differently.

Version 4.8.1's `_transfer` function holds a `_beforeTokenTransfer` hook which can be overriden to fit in any other special characteristics, including the blacklisting feature, which was done in the [previous](https://github.com/sherlock-audit/2023-02-telcoin/blob/97b2bcd78740a870159b25d2f04806068c1c4c1f/telcoin-audit/contracts/stablecoin/Stablecoin.sol#L192) StableCoin contract. Version 5.0.1's `_transfer` function lacks the hook and as evident from the current StableCoin contract, no modifiers or form of _transfer/_update override is implemented to prevent blocklisted users from sending or receiving tokens. Thus the blocklist feature is left partially redundant, with the only possible cost being the loss of user's tokens upon being blocklisted.

Taking a page from an [issue](https://github.com/sherlock-audit/2023-02-telcoin-judging/issues/43), branded won't fix in the previous audit of the StableCoin contract, however this time with a little bit more consequence, a malicious user viewing the mempool can frontrun calls to the `addBlacklist` function, to transfer the tokens out of his address to another address, but unlike the previous issue, due to the ineffective blocklisting, can later receive the tokens and continue interacting with the protocol as normal. Efforts to re-blocklist him will fail unless he's first removed from blocklist, upon which he can always repeat the frontrun of the `addBlacklist` process again.

## Impact

Blocklisting is essentially ineffective and users can evade the loss of funds by frontrunning the blocklist and sending their tokens to another address, returning it to their main and continue interacting with the protocol as normal.

## Code Snippet
https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/21920190e0772afa18e7f856a036fea3ef5b9635/telcoin-contracts/contracts/util/abstract/Blacklist.sol#L72-L79
https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/stablecoin/Stablecoin.sol

## Tool used
Manual Code Review

## Recommendation
1. Use an older version of openzeppelin contracts that implements the `_beforeTokenTransfer` hook, and implement the same override as done in the previous StableCoin contract.
2. Implement the token transfer functions and a check or modifiers for blocklisted users.