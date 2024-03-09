Cheery Rainbow Shetland

high

# `BridgeRelay` makes an approval to the same `predicate` address, despite different tokens/ eth having different predicates.

## Summary
`BridgeRelay` makes an approval to the same `predicate` address, despite different tokens/ eth having different predicates.

## Vulnerability Detail
Let's look at how `BrtidgeRelay`  bridges erc20s to Polygon 
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
As it can be seen, despite what the token is, it always makes the approval to the constant `PREDICATE_ADDRESS`. 

However, let's take a look at the `_depositFor` function within the bridge.

```solidity

    function _depositFor(
        address user,
        address rootToken,
        bytes memory depositData
    ) private {
        bytes32 tokenType = tokenToType[rootToken];
        require(
            rootToChildToken[rootToken] != address(0x0) &&
               tokenType != 0,
            "RootChainManager: TOKEN_NOT_MAPPED"
        );
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
There are 2 main things to point out - there's a `tokenToType` mapping and a `typeToPredicate` mapping. Meaning that different tokens/ eth have different predicate addresses. 

Assuming that all tokens within `BridgeRelay` will use that same `predicate` is wrong also because it can also be changed at any time for any token.

## Impact
Contract will be unusable for multiple tokens and might become also unusable for any/every token

## Code Snippet
https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/bridge/BridgeRelay.sol#L67C1-L81C6
https://etherscan.io/address/0x37d26dc2890b35924b40574bac10552794771997#code#F1#L2201

## Tool used

Manual Review

## Recommendation
Before making the approval make a check to the POS Bridge's `tokenToType` and `typeToPredicate` to get the predicate and then approve it.
