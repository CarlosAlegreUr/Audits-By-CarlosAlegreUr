### Summary

If the winner is a smart contract implementing account abstraction, it might not be able to claim the prize as they might not own the same address in Ethereum mainnet. 


### Root Cause

Account abstraction wallets can have different addresses in different chains. A smart contract address implementing account abstraction in Avalanche might not be owned by the same person in Ethereum mainnet.

### Internal pre-conditions

Does not apply.

### External pre-conditions

Does not apply.

### Attack Path

1. _AA_ (Account Abstraction) wallet buys points and they are minted to `msg.sender`.
2. _AA_ wallet wins a prize.
3. The prize is `propagateRaffleWinner()` which will use as winner the `msg.sender` saved on `_ticketOwnership` map on the `WinnablesTicket` contract to set that address as the winner on Ethereum through CCIP. 
4. CCIP is successful and the winner is set on Ethereum. However that address might be owned by no-one or someone else.

### Impact

Someone else wins the lottery or even worse funds are stuck there forever until someone manages to figure out a way of getting that address in Ethereum. This can be forever.

### PoC

The winner in `propagateRaffleWinner()` is taken from the `_ticketOwnership` map on the `WinnablesTicket` contract trhough the `ownerOf()`. See [here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicketManager.sol#L338), then [here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicketManager.sol#L476), and then [here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicket.sol#L100).

This value was set as `msg.sender` when minting the tokens through the `buyTickets()` [here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicketManager.sol#L208). See minting function [here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicket.sol#L194).

### Mitigation

Allow the winner to chose a destination address before propagating.