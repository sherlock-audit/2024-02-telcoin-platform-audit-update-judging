Exotic Concrete Boa

medium

# Not all ERC20 tokens can be bridged because of hardcoded `PREDICATE_ADDRESS`

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
