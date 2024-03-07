Obedient Navy Jellyfish

high

# Blacklisted User can still interact with Stablecoin

## Summary
The `Stablecoin` contract intends to have a blacklisting functionality built in. Upon blacklisting a user, all of their tokens are transferred to the blacklister. But, the implementation still allows blacklisted users to get tokens minted to, tokens can be transferred to/from the blacklisted user, and basically all the ERC20 functionality will still work for the blacklisted user, thereby allowing them to transact using the blacklisted token.

## Links to affected code

https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/stablecoin/Stablecoin.sol

https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/util/abstract/Blacklist.sol#L72

## Vulnerability Detail
The `Stablecoin` contract inherits from `Blacklist` contract to support blacklisting. Also the contest page states that:

```txt
Do you expect to use any of the following tokens with non-standard behaviour with the smart contracts?

Blacklisting on the stablecoin contract.
```

Furthermore, the [`Stablecoin` README](https://github.com/sherlock-audit/2024-02-telcoin-platform-audit-update/blob/main/telcoin-contracts/contracts/stablecoin/README.md) says:

```txt
## Contracts

### `Stablecoin`

.......built-in mechanism for blacklisting addresses to prevent misuse.

#### Key Stablecoin Features

...
...

Blacklisting functionality to restrict transactions from specific addresses.

...
...
```

However, it doesn't work as expected. Once a user is blacklisted, then their tokens are transferred to the caller with the `BLACKLISTER_ROLE` role.

But, the blacklisted user can still get tokens minted to themselves, transferred from/to other users, and operate with the `Stablecoin` token as a normal user can.

## Impact
Based on the documentation, the protocol specifies that there is inbuild mechanism for backlisting, and that blackliste users are restricted from transacting using this token.

But as stated in the description, this is not true, and the blacklisted user can normally interact with the `Stablecoin`.

## Code Snippet
Place the below code in `test/stablecoins/Stablecoin.test.ts`:

```javascript
describe("Stablecoin - Blacklisted User", () => {
    let deployer: SignerWithAddress;
    let blacklistedUser: SignerWithAddress;
    let validHolder: SignerWithAddress;

    beforeEach("Setup", async () => {
        [deployer, blacklistedUser, validHolder,] = await ethers.getSigners();

        await stablecoin.grantRole(BLACKLISTER_ROLE, deployer);
        await stablecoin.grantRole(MINTER_ROLE, deployer);
    });

    it("Stablecoin can be minted to Blacklisted user", async () => {
        // All balances start with 0
        expect(await stablecoin.balanceOf(blacklistedUser)).to.equal(0);
        expect(await stablecoin.balanceOf(deployer)).to.equal(0);
        expect(await stablecoin.balanceOf(validHolder)).to.equal(0);

        await stablecoin.connect(deployer).mintTo(blacklistedUser, 100);
        // Blacklisted User gets 100 tokens minted before getting blacklisted
        expect(await stablecoin.balanceOf(blacklistedUser)).to.equal(100);

        await stablecoin.connect(deployer).addBlackList(blacklistedUser);
        // The balance of Blacklisted User (100 tokens)
        // gets transferred to the caller
        // and then gets blacklisted
        expect(await stablecoin.balanceOf(blacklistedUser)).to.equal(0);
        expect(await stablecoin.balanceOf(deployer)).to.equal(100);
        expect(await stablecoin.balanceOf(validHolder)).to.equal(0);

        // The blacklisted user still gets the token minted
        await stablecoin.connect(deployer).mintTo(blacklistedUser, 100);
        expect(await stablecoin.balanceOf(blacklistedUser)).to.equal(100);
        expect(await stablecoin.balanceOf(deployer)).to.equal(100);
        expect(await stablecoin.balanceOf(validHolder)).to.equal(0);

        // The blacklisted user can transfer tokens too, and interact normally
        await stablecoin.connect(blacklistedUser).transfer(validHolder, 50);
        expect(await stablecoin.balanceOf(blacklistedUser)).to.equal(50);
        expect(await stablecoin.balanceOf(deployer)).to.equal(100);
        expect(await stablecoin.balanceOf(validHolder)).to.equal(50);
    });

    it("Blacklisted user can get stablecoin from other users and use it", async () => {
        // All balances start with 0
        expect(await stablecoin.balanceOf(blacklistedUser)).to.equal(0);
        expect(await stablecoin.balanceOf(deployer)).to.equal(0);
        expect(await stablecoin.balanceOf(validHolder)).to.equal(0);

        // User gets blacklisted
        await stablecoin.connect(deployer).addBlackList(blacklistedUser);

        // A valid holder transfers tokens to blacklisted user
        await stablecoin.connect(deployer).mintTo(validHolder, 100);
        await stablecoin.connect(validHolder).transfer(blacklistedUser, 50);
        // Blacklisted User gets 50 tokens trasnferred to them

        expect(await stablecoin.balanceOf(blacklistedUser)).to.equal(50);
        expect(await stablecoin.balanceOf(deployer)).to.equal(0);
        expect(await stablecoin.balanceOf(validHolder)).to.equal(50);
    });
});
```

## Tool used

Manual Review

## Recommendation

For all the ERC20 methods, add checks to ensure that the address is not blacklisted, so that the ERC20 methods do not work for the blacklisted addresses.