Passive Cobalt Unicorn

high

# Incorrect Implementation of Blacklist

## Summary

The documentation states that "Blacklisting has been included to prevent this currency from being used for illicit or nefarious activities." However, a blacklisted user can still transfer the subjected ERC20 Stablecoin.sol.

## Vulnerability Detail

## Impact

The parent contract of Stablecoin.sol, Blacklist.sol, is an abstract contract that provides functions to add and remove users from the blacklist. However, there is no implementation to prevent blacklisted users from using the token. This oversight can lead to unintended behaviors, such as allowing malicious third parties to continue using the service provided by the protocol, even after their address has been added to the blacklist.

## Code Snippet

https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/util/abstract/Blacklist.sol#L62-L65

## Tool used

Manual Review

## Recommendation

Consider adding a hook like the one below and override the functions that the protocol wants to prevent the blacklisted user from using in Stablecoin.sol:

```solidity
  function _beforeTokenTransfer(address from, address to) internal virtual view {
        require(!isBlackListed(msg.sender), "MSG_SENDER_IS_BLACKLISTED_BY_TOKEN_OWNER");
        require(!isBlackListed(from), "FROM_IS_BLACKLISTED_BY_TOKEN_OWNER");
        require(!isBlackListed(to), "TO_IS_BLACKLISTED_BY_TOKEN_OWNER");
    }
```
