### Summary

Anyone can cancel a raffle with `PRIZE_LOCKED` status. This creates an easy way to stop the protocol from creating any raffle via front-running.

### Root Cause

`cancelRaffle()` function at `WinnableTicketManager` can be called by anyone on raffles with the `PRIZE_LOCKED` state.

### Internal pre-conditions

Does not apply.

### External pre-conditions

Does not apply.

### Attack Path

1. Admin locks prize on Ethereum mainnet. Any `lock()` func that trigges a `ccipSend()`.
2. After CCIP is completed CCIP router ends up calling `_ccipReceive()`, setting the raffle state to `PRIZE_LOCKED`
3. Admins, now in Avalanche, call `createRaffle()` yet malicious attacker front-runs them and  calls `cancelRaffle()`.
4. It is true that the locked prize will be unlocked in ethereum for the admins again, but no-raflle will be ever carried out.

### Impact

Cretaion of raffles can be easily DOS by anyone. As the protocol implements a simple mechanism for raffles, other competitors in the raffle business or really anyone with malicious intentions on the space can leverage this to completely DOS the protocol as no raffle won't be ever started.

### PoC

As you can see in `cancelRaffle()` at `WinnableTicketManager` there is this check: `_checkShouldCancel()`. If the state is `PRIZE_LOCKED` this check will pass. See [here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicketManager.sol#L436). Also `cancelRaffle()` can be called by anyone. See [here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicketManager.sol#L278), no `onlyRole(X)` modifier nor any revert related to `msg.sender` along its code.

`PRIZE_LOCKED` is only set when a cross-chain tx arrives and calls `_ccipReceive()`. See [here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicketManager.sol#L381). After that to create the raffles the admins call `createRaffle()` and set the status to `IDLE`. See [here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicketManager.sol#L264). But anyone can front-run this call and call `cancelRaffle()` with the `raffleId` and the raffle will be cancelled.

Notice that this can't be fixed with a multicall as `_ccipReceive()` can only be called by the CCIP router. See [here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/BaseCCIPReceiver.sol#L34). So the protocol must always backrun this tx to create a raffle, yet this opens the posibilty for anyone to front-run the creation of raffles and cancel them.

```solidity
function _checkShouldCancel(uint256 raffleId) internal view {
    Raffle storage raffle = _raffles[raffleId];
    // 1️⃣🔽🟢🔽 SEE HERE, if raffle PRIZE_LOCKED the check passes and raffle is cancelled 
    if (raffle.status == RaffleStatus.PRIZE_LOCKED) return; 
    if (raffle.status != RaffleStatus.IDLE) revert InvalidRaffle();
    if (raffle.endsAt > block.timestamp) revert RaffleIsStillOpen();
    uint256 supply = IWinnablesTicket(TICKETS_CONTRACT).supplyOf(raffleId);
    if (supply > raffle.minTicketsThreshold) revert TargetTicketsReached();
}

// 2️⃣🔽🟢🔽 SEE HERE, cancelRaffle() can be called by anyone
function cancelRaffle(address prizeManager, uint64 chainSelector, uint256 raffleId) external {
    _checkShouldCancel(raffleId);
    // 3️⃣🔽🟢🔽 SEE HERE, raffle cancelled
    _raffles[raffleId].status = RaffleStatus.CANCELED;
    _sendCCIPMessage(
        prizeManager, chainSelector, abi.encodePacked(uint8(CCIPMessageType.RAFFLE_CANCELED), raffleId)
    );
    IWinnablesTicket(TICKETS_CONTRACT).refreshMetadata(raffleId);
}
```

### Mitigation

If the raffle status is `PRIZE_LOCKED` and the caller of `cancelRaffle()` is not the admin (`onlyRole(0)`), the function must revert.