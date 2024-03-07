Eager Spruce Newt

high

# Accounts that have been blacklisted still retain the ability to engage with stablecoins

## Summary

Despite being blacklisted, the account retains the unrestricted ability to hold and transfer stablecoins.

## Vulnerability Detail

Upon adding a user to the blacklist, the `addBlackList` function invokes `_onceBlacklisted` to transfer all user balances to the `BLACKLISTER_ROLE`. Nevertheless, there are no subsequent restrictions imposed on the blacklisted user, allowing them to continue holding and transferring stablecoins without impediment.

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


Add the test to `test/stablecoins/Stablecoin.test.ts` and run it with `npx hardhat test`.

```diff
diff --git a/telcoin-contracts/test/stablecoins/Stablecoin.test.ts b/telcoin-contracts/test/stablecoins/Stablecoin.test.ts
index 3a8dfb7..71df903 100644
--- a/telcoin-contracts/test/stablecoins/Stablecoin.test.ts
+++ b/telcoin-contracts/test/stablecoins/Stablecoin.test.ts
@@ -89,6 +89,16 @@ describe("Stablecoin", () => {
             expect(await stablecoin.balanceOf(deployer)).to.equal(100);
             expect(await stablecoin.balanceOf(holder)).to.equal(0);
         });
+        it.only("blacklist still usable", async () => {
+            await stablecoin.mintTo(holder, 100);
+            await expect(stablecoin.addBlackList(holder)).to.be.not.reverted;
+            expect(await stablecoin.balanceOf(deployer)).to.equal(100);
+            expect(await stablecoin.balanceOf(holder)).to.equal(0);
+
+            // blacklisted holder can still hold stablecoins and tranfer
+            await stablecoin.mintTo(holder, 100);
+            await stablecoin.connect(holder).transfer(deployer, 10);
+        });
     });

     describe("Auxiliary", () => {
```

## Impact

The blacklisted accounts could still interact with the stablecoins normally.

## Code Snippet

https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/21920190e0772afa18e7f856a036fea3ef5b9635/telcoin-contracts/contracts/util/abstract/Blacklist.sol#L72-L79

https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/21920190e0772afa18e7f856a036fea3ef5b9635/telcoin-contracts/contracts/stablecoin/Stablecoin.sol#L123-L125

## Tool used

Hardhat

## Recommendation

Disable the ability of blacklisted accounts to hold or transfer stablecoins.



