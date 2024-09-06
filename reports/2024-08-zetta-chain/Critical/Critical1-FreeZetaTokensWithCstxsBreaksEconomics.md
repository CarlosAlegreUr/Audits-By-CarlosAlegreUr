## Relevant Context

`FUNGIBLE_MODULE` is a special address at protocol level in Zeta that does not pay gas. Devs confirmed this in a question I made them, see picture attached (question number **2**) or Cantina question [here](https://cantina.xyz/code/80a33cf0-ad69-4163-a269-d27756aacb5e/comments#comment-a8ad839a-d7fd-476e-a6fb-45e98d2b79dd) and answer [here](https://cantina.xyz/code/80a33cf0-ad69-4163-a269-d27756aacb5e/comments#comment-2ae513c0-6910-4e11-9442-d8689469eac9).

![Screenshot from 2024-08-24 11-52-05.png](https://imagedelivery.net/wtv4_V7VzVsxpAFaxzmpbw/276a062b-2392-44cc-478a-39482199fd00/public) 

## Description

When sending transaction cross-chain from spoke chain to Zeta chain no fees are charged beacause in Zeta chain it is free to execute the txs due to the priviliged address `FUNGIBLE_MODULE` at protocol level that calls the execution and this address does not pay gas.

Yet this creates 3 critical issues:

- Why would users ever use Zeta chain nodes to execute transactions if they can do it for free from spoke chains with lower gas costs?

- Users can DOS the `Zeta` chain. Average gas transaction fee on `Solana` is [0.00015 SOL](https://coincheckup.com/blog/solana-gas-fee/) = 0.02 USD. Users can deploy a smart contract on `Zeta` chain with a function that needs gas to be executed close to the `block gas limit` and call it for just 0.02 USD. A big spam of this transaction could lead to a temporary DOS of the whole chain.

- Users can execute stuff for free from Zeta too. They just gotta send a cross-chain tx from `Zeta -> Spoke` chain and make the destination chain revert. Then on Zeta the `onRevert()` hook will be called by the `FUNGIBLE_MODULE`, effectively executing gas-free whatever you want. Just for the cost of the first small cross-chain transaction.

## Impact Explanation

Users can call gas heavy functions on the `Zeta` chain for a negligible fraction of the execution cost. This can lead to a lot of severe consequences like DOS attack, loss of sovereignty and an unusable blockchain.

## Recommendation

The protocol must charge fees to users according to gas usage in any direction, even if it is technically free for the protocol to execute them when destination is Zeta chain.

A way of doing this is with the already used `gasLimit` based fees used when `Zeta-to-SpokeChain` operations are carried out.
