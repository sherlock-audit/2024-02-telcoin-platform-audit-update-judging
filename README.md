# Issue M-1: Blacklisted accounts can still transact. 

Source: https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update-judging/issues/4 

## Found by 
0xkmg, Krace, Tendency, ZanyBonzy, ZdravkoHr., blutorque, bughuntoor, cawfree, merlin, neocrao, sa9933, smbv-1923, turvec
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



## Discussion

**sherlock-admin4**

1 comment(s) were left on this issue during the judging contest.

**takarez** commented:
>  valid; high(1)



**sherlock-admin4**

The protocol team fixed this issue in PR/commit https://github.com/telcoin/telcoin-contracts/pull/3.

# Issue M-2: Not all ERC20 tokens can be bridged because of hardcoded `PREDICATE_ADDRESS` 

Source: https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update-judging/issues/26 

## Found by 
ZdravkoHr., bughuntoor
## Summary
`PREDICATE_ADDRESS` in [`BridgeRelay.sol`](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/21920190e0772afa18e7f856a036fea3ef5b9635/telcoin-contracts/contracts/bridge/BridgeRelay.sol#L32) is hardcoded to the [`ERC20Predicate`](https://github.com/maticnetwork/pos-portal/blob/master/contracts/root/TokenPredicates/ERC20Predicate.sol). This means that any tokens that use other predicates will not be bridgeable.

## Vulnerability Detail
When the [`BridgeRelay.transferERCToBridge`](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/21920190e0772afa18e7f856a036fea3ef5b9635/telcoin-contracts/contracts/bridge/BridgeRelay.sol#L67-L81) gets called, it approves the hardcoded `ERC20Predicate` to use the ERC20 tokens. However, the bridge uses more than one predicate to lock the tokens. [Here](https://github.com/maticnetwork/pos-portal/blob/d6c21182b1c379a858cf4cf23d34837b199e4908/contracts/root/RootChainManager/RootChainManager.sol#L302) the bridge retrieves the predicate address based on the type of token to be bridged. If a token that uses different predicate than the hardcoded is sent to the `BridgeRelay`, it will be forever stuck there since the right predicate will not have approval to transfer it. There also exists a risk that a token can change its predicate at any time.

## Impact
Tokens that use different predicate will be forever stuck in the contract.

## Code Snippet
```solidity
    function transferERCToBridge(IERC20 token) internal {
        //zero out approvals
        token.forceApprove(PREDICATE_ADDRESS, 0);
        // increase approval to necessary amount
        token.safeIncreaseAllowance(
            PREDICATE_ADDRESS,
            token.balanceOf(address(this))
        );
        //deposit
        POS_BRIDGE.depositFor(
            address(this),
            address(token),
            abi.encodePacked(token.balanceOf(address(this)))
        );
    }
```

```solidity
        address predicateAddress = typeToPredicate[tokenType];
        require(
            predicateAddress != address(0),
            "RootChainManager: INVALID_TOKEN_TYPE"
        );
        require(
            user != address(0),
            "RootChainManager: INVALID_USER"
        );

        ITokenPredicate(predicateAddress).lockTokens(
            _msgSender(),
            user,
            rootToken,
            depositData
        );
```
## Tool used

Manual Review

## Recommendation
Instead of hardcoding the predicate, query the bridge when transferring the ERC20 token.
```diff
    function transferERCToBridge(IERC20 token) internal {
+      bytes32 tokenType = POS_BRIDGE.tokenToType(address(token));
+      bytes32 predicateAddress= POS_BRIDGE.typeToPredicate(tokenType);
        //zero out approvals
        token.forceApprove(PREDICATE_ADDRESS, 0);
        // increase approval to necessary amount
        token.safeIncreaseAllowance(
-           PREDICATE_ADDRESS,
+           predicateAddress
            token.balanceOf(address(this))
        );
        //deposit
        POS_BRIDGE.depositFor(
            address(this),
            address(token),
            abi.encodePacked(token.balanceOf(address(this)))
        );
    }
```



## Discussion

**sherlock-admin3**

1 comment(s) were left on this issue during the judging contest.

**takarez** commented:
>  invalid



**amshirif**

This isnt an issue. So we dont provide bridging as a service. We use it as a recovery method for people who send their tokens on the wrong network as a means of recovering them. However, tokens that we dont support we dont do anything with. We just allow them to sit there. The reason being is that if we were to bridging them over they still couldnt send them on our platform. We dont support and bridged tokens that us different predicates. Whats more, since we'd have to provide information in such as which bridge to use, it would require it to be a privileged method, which is something we're actively avoiding. We wish the function to be callable to anyone who wishes to attempt to push their tokens across.

**WangSecurity**

After additional discussion we came to the conclusion that this issue is valid, cause the predicated address may be changed by polygon at any moment and users funds will be lost with to method to recover them.
Therefore, it is a Medium.

**sherlock-admin4**

The protocol team fixed this issue in PR/commit https://github.com/telcoin/telcoin-contracts/pull/8.

