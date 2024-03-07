Cheery Rainbow Shetland

medium

# Users can still receive and send tokens after getting blacklisted

## Summary
Users can still receive and send tokens after getting blacklisted 

## Vulnerability Detail
The `stablecoin` contracts inherit blacklisting mechanism. Although upon getting `blacklisted`, the user's funds are transferred, the user can still receive and send tokens, since non of the transferring methods are overridden. 

```solidity
    function addBlackList(
        address user
    ) public virtual onlyRole(BLACKLISTER_ROLE) {
        if (blacklisted(user)) revert AlreadyBlacklisted(user);
        _setBlacklist(user, true);
        _onceBlacklisted(user);
        emit AddedBlacklist(user);
    }
```

The readme states that blacklisted users should not be able to transfer tokens: 

> Blacklisting functionality to restrict transactions from specific addresses.


## Impact
Blacklisting mechanism is supposed to restrict people from sending transactions, but doesnt actually do so

## Code Snippet

## Tool used
https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/util/abstract/Blacklist.sol#L72C1-L79C6

Manual Review

## Recommendation
override the transfer methods to not allow transfers between blacklisted accounts
