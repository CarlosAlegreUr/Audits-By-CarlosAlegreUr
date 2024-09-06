## Description

In chains where Zeta token is not native and thus `ZetaConnectorNonNative` is used, anyone can burn their own tokens.

## Impact Explanation

If they burn them they will never be able to recover them, and the total supply will be reduced. This completely breaks the tokenomics of the essential and core Zeta token.

If people burn tokens the team can just mint them back using the `TSS` to mint those tokens back with the `withdraw()` function. But this is clearly not the expected use of `withdraw()`, they have been lucky to have a way of minting people tokens back. Tokenomics are broken, people are not supposed to be able to burn their tokens and the `TSS` is not supposed at all to mint people tokens back using the `withdraw()` function. The purpose of that function is to mint tokens to people that initiated cross-chain transactions from Zeta chain.

Allowing people to burn their tokens has unexpected consequences (yet temporarily fixable until someone else decides to burn and we got the problem again) that break the expected tokenomics of the Zeta token and require costly constant monitoring and manual intervention to "fix" every time.

## Proof of Concept

To burn their tokens people can just call `receiveTokens()` at `ZetaConnectorNonNative.sol`:

```solidity
 // ⏬👁️🟢 No access control modifier
 function receiveTokens(uint256 amount) external override whenNotPaused {
        // ⏬👁️🟢 It just burns the tokens, not even minting new ones to address(this)
        // ⏬👁️🟢 to maintain the totalsupply.
        IZetaNonEthNew(zetaToken).burnFrom(msg.sender, amount);
    }
```

I think the reason behind not minting to `address(this)` is because the devs assumed `receiveTokens()` will be called by the `GatewayEVM` contract which correctly signals the `Observer`, right after the burn, to mint new tokens in the Zeta chain.

Yet anyone can call this function, burn their tokens and mint no new tokens anywhere manipulating the `totalSupply`.

## Recommendation

Add an `onlyGateway` modifier or mint new tokens to `address(this)` after the burn.

> 🚧 **Note** ⚠️ I'm not sure about minting to `address(this)` is the safest option so I encourage just restricting the call acces to `receiveTokens()`.
