### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

## Summary

**VVV** token could be maliciously sent to the sytem's contracts and these tokens would remain stuck forever as there is no way of transfering them out.

## Vulnerability Detail

No way of transfering out **VVV** in the system contracts: `VVVETHStaking`, `VVVAuthorizationRegistry` and `VVVVesting`.

The only way where **VVV** token is transfered out of the `VVVETHStaking` contract is inside the `claimVvv()` function. But the very same contract can't call this function from itself, and even if it could, in order to use it it would require to stake ETH on itself.

Users can send **VVV** tokens to the systems contracts on purpose so the token gets stuck there forever. Influencing in unexpected ways the tokenomics. This could be with directly `transfers()` or in a more camouflaged way using the:

```solidity
        (bool withdrawSuccess,) = payable(msg.sender).call{value: stake.stakedEthAmount}("");
```
When `withdrawStake()` is called. For example `msg.sender` would be a smart contract with a `receive()` function that when called transfers **VVV** tokens to some systems contract, for example the very same `VVVETHStaking`.

The system can't do anything against people sending tokens to random dead addresses or smart contracts with no transfering capabilities. But could at least protect itself from tokens stuck on their own smart contracts.

## Impact

People could puporsely leave forever **VVV** tokens out of circulation. And as the token has a cap limit the circulating supply and tokenomics could be affected and manipulated in unexpected ways. 

## Code Snippet

The only transfer of **VVV** token is inside the `claimVvv()`.
See [here](https://github.com/sherlock-audit/2024-03-vvv-vesting-staking/blob/main/vvv-platform-smart-contracts/contracts/staking/VVVETHStaking.sol#L191).

## Tool used

Manual Review

## Recommendation

As there is a function for the protocol to `withdrawEth()` from the `VVVETHStaking` contract thus they already hold that much `TRUSTED` power. I see fit and necessary having the same functonality for their own **VVV** token.

All system contracts should have this withdraw function for **VVV** tokens.
