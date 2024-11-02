## Vulnerability Details

As per Chainlink Staking V0.2 official instructions, see [here](https://ipfs.io/ipfs/QmUWDupeN4D5vHNWH6dEbNuoiZz9bnbqTHw61L27RG6tE2) or [here](https://github.com/smartcontractkit/chainlink-staking-v0.2-public-guide/blob/main/instructions.txt).

To stake you have to transfer LINK token to the staking pool with a `transferAndCall()` which its data bytes argument holding the following value:

`0x00000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000000`

Regardless of the staking pool, operator or community.

The code sends an empty bytes value as we can see in the `Vault.sol` deposit function [here](https://github.com/stakedotlink/contracts/blob/native-link-withdrawals/contracts/linkStaking/base/Vault.sol#L69):

```solidity
IERC677(address(token)).transferAndCall(address(stakeController), _amount, ""); 
``` 

## Impact

I've been looking at the Chainlink Pool's code and I could not find any bad consequence. I have no idea why they emphasize that you have to pass those bytes.

Only consequence might come when the Chainlink whitelist was still running but it is already closed and the staking is already live for everyone.

## Recommendations

Ask the Chainlink Stakig devs why do you require those bytes and if it is truly necessary add them to the code. 

