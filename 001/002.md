Exotic Concrete Boa

high

# Blacklisting functionality can be completely bypassed by sandwiching the transaction

## Summary
The [Stablecoin](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/21920190e0772afa18e7f856a036fea3ef5b9635/telcoin-contracts/contracts/stablecoin/Stablecoin.sol#L19) contract inherits from [Blacklist](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/21920190e0772afa18e7f856a036fea3ef5b9635/telcoin-contracts/contracts/util/abstract/Blacklist.sol) to allow the owner of the protocol to stop certain addresses from committing nefarious actions. However, the standard ERC20 `transfer` function does not take into account whether the addresses it operates with are blacklisted or not. This can render the whole blacklisting functionality useless.

## Vulnerability Detail
In order to add an address to the blacklist, someone holding the `BLACKLISTER_ROLE` has to call [Blacklist.addBlackList()](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/21920190e0772afa18e7f856a036fea3ef5b9635/telcoin-contracts/contracts/util/abstract/Blacklist.sol#L72-L79). Then [Stablecoin._onceBlacklisted](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/21920190e0772afa18e7f856a036fea3ef5b9635/telcoin-contracts/contracts/stablecoin/Stablecoin.sol#L124) will be called. It will transfer all tokens of the address to be blacklisted to the one who is performing the blacklist operation. This looks like a fine blacklisting functionality, but it can be easily bypassed because the `transfer` function does not know which addresses are blacklisted.

#### Exploit steps:
1. `BLACKLISTER` initiates a transaction to add Bob to the blacklist.
2. Bob frontruns the transaction and sends all his balance to his second account (Bob2)
3. The blacklisting transaction gets executed, Bob is added to the blacklist and his balance (0) is being transferred to the `BLACKLISTER`.
4. Bob2 backruns the transaction and sends all the tokens back to Bob.
5. Now Bob can continue to operate freely with his tokens because the standard ERC20 actions are not gated for blacklisted addresses.

#### POC:
Add the following `it` to this [describe](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/21920190e0772afa18e7f856a036fea3ef5b9635/telcoin-contracts/test/stablecoins/Stablecoin.test.ts#L80-L92)
```javascript
        it ("sandwich", async () => {
            const [,,holder2] = await ethers.getSigners();
            await stablecoin.mintTo(holder, 100);

            // Sandwich the transaction to avoid transferring tokens
            await stablecoin.connect(holder).transfer(holder2.address, 100);
            await stablecoin.addBlackList(holder);
            await stablecoin.connect(holder2).transfer(holder.address, 100);

            // The blacklisted user can continue to use the tokens as if they were never blacklisted
            await expect(
              await stablecoin.connect(holder).transfer(holder2.address, 100)
            ).to.be.not.reverted;
        });
```

## Impact
The blacklisting functionality can be easily bypassed letting malicious addresses operate freely with the stablecoin.

## Code Snippet
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

```solidity
    function _onceBlacklisted(address user) internal override {
        _transfer(user, _msgSender(), balanceOf(user));
    }
```

## Tool used
Hardhat

## Recommendation
Override the transfer functions and make them revert if they initiate a transfer from a blacklisted user.
