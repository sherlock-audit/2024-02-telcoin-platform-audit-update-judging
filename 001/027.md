Sour Chiffon Shark

medium

# The `Stablecoin` smart contract does not prevent blacklisted addresses from interacting with it

## Summary

`Blacklist` smart contract is used for the prevention of the interaction of certain addresses, which is not actually implemented.

## Vulnerability Detail

If a wallet with the `BLACKLISTER_ROLE` role adds a user to the blacklist, then all of the user's balance is transferred to the caller.

```solidity
function addBlackList(
        address user
    ) public virtual onlyRole(BLACKLISTER_ROLE) {
        if (blacklisted(user)) revert AlreadyBlacklisted(user);
        _setBlacklist(user, true);
-->     _onceBlacklisted(user);
        emit AddedBlacklist(user);
    }
```
```solidity
function _onceBlacklisted(address user) internal override {
-->     _transfer(user, _msgSender(), balanceOf(user));
    }
```
After being blacklisted, the user can continue to interact with the `Stablecoin` smart contract by sending and receiving tokens.

## Impact
The `Stablecoin` smart contract does not prevent blacklisted addresses from interacting with it.

## Code Snippet
[contracts/util/abstract/Blacklist.sol#L77](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/util/abstract/Blacklist.sol#L77)
[contracts/stablecoin/Stablecoin.sol#L123-L125](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/stablecoin/Stablecoin.sol#L123-L125)

## Tool used

Manual Review

## Recommendation
Consider overriding the `_update` function from `OpenZeppelin` for any customizations related to transfers, mints, and burns, as per the contract's design.

```diff
+function _update(address from, address to, uint256 value) internal override {
+        require(!blacklisted(from), "Stablecoin: from cannot be blacklisted address");
+       require(!blacklisted(to), "Stablecoin: to cannot be blacklisted address");
+      super._update(from,to,value);
+    }
```

