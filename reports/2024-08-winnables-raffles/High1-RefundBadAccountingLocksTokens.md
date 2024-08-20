### Summary

The refund mechanism in case of cancelled raffle has bad accounting which leads to funds stuck.

### Root Cause

At `refundPlayers()` funds previously accounted as `_lockedETH` are not substracted from the state when withdrawn from the system.

### Internal pre-conditions

Does not apply.

### External pre-conditions

Does not apply.

### Attack Path

1. Admin creates a raffle.
2. Anyone participate via `buyTickets()` and `msg.value` sent is locked from withdrawals at `_lockedETH`. (lets say `msg.value = 10`) 
3. Raffle ends up being cancelled for any valid reasons, like minimum tickets not reached.
4. Client calls `refundPlayers()` to get the funds back. He gets them, balance of contract == 0.
5. Yet the `_lockedETH` value is the same.
6. Another raffle is created.
7. A bucnh of people call `buyTickets()` and send `msg.value` to the contract. Increase by 50 lets say. Total value earned by the raffle to the protocol is 50. `TotalRaised==50, _lockedETH==10+50==60, address(this).balance==50`.
8. Raffle ends and `propagateRaffleWinner()` is called, unlocking raised funds. `_lockedETH==60-50==10`.
9. Admin calls `withdrawETH()`, and `balance=address(this).balance - _lockedETH` is calculated. `balance==50-10==40`. 40 is withdrawn yet the value raised is 50 and all of it should be withdrawable.

### Impact

Every refunded amount will end up stucking in the contract that amount and actually the loss is taken by the protocol from future raffles revenue.

### PoC

`_lockedETH` is only altered in 3 parts of the code: `buyTickets()` ([here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicketManager.sol#L206)), `propagateRaffleWinner()` ([here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicketManager.sol#L343)), `withdrawETH()` ([here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicketManager.sol#L303)).

Follow the attack path calls and see the links to easily see the execution path is correct.

I will add a PoC executable code if I got enough time later.

### Mitigation

Substract at `refundPlayers()` the amounts refunded from the `_lockedETH` state.