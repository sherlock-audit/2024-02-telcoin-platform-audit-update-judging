Acrobatic Eggshell Snail

medium

# Blacklisted accounts can still transact.

## Summary

Accounts that have been blacklisted by the [`BLACKLISTER_ROLE`](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/21920190e0772afa18e7f856a036fea3ef5b9635/telcoin-contracts/contracts/util/abstract/Blacklist.sol#L32) continue to transact normally.

## Vulnerability Detail

Currently, the only real effect of blacklisting an account is the seizure of [`Stablecoin`](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/stablecoin/Stablecoin.sol) funds:

```solidity
/**
 * @notice Overrides Blacklist function to transfer balance of a blacklisted user to the caller.
 * @dev This function is called internally when an account is blacklisted.
 * @param user The blacklisted user whose balance will be transferred.
 */
function _onceBlacklisted(address user) internal override {
  _transfer(user, _msgSender(), balanceOf(user));
}
```

However, following a call to [`addBlackList(address)`](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/21920190e0772afa18e7f856a036fea3ef5b9635/telcoin-contracts/contracts/util/abstract/Blacklist.sol#L72C14-L72C26), the blacklisted account may continue to transact using [`Stablecoin`](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/stablecoin/Stablecoin.sol).

Combined with previous audit reports, which attest to the blacklist function's [susceptibility to frontrunning](https://github.com/sherlock-audit/2023-02-telcoin-judging/issues/43), the current implementation of the blacklist operation can effectively be considered a no-op.

## Impact

Medium, as this the failure of a manually administered security feature.

## Code Snippet

### [📄 Stablecoin.sol](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/stablecoin/Stablecoin.sol)

## Tool used

Manual Review

## Recommendation

ERC20s that enforce blacklists normally prevent a sanctioned address from being able to transact:

### [📄 Stablecoin.sol](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/stablecoin/Stablecoin.sol)

```diff
+ error Blacklisted(address account);

+function _update(address from, address to, uint256 value) internal virtual override {
+
+  if (blacklisted(from)) revert Blacklisted(from); 
+  if (blacklisted(to)) revert Blacklisted(to);
+
+  super._update(from, to, value);
+}
```
