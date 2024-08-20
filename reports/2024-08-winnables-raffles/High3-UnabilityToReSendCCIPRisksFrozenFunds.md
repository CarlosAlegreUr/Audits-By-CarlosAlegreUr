### Summary

If a re-org occures in desitantion chain after the `Commitment-DON` commits, the `RMN` network will curse CCIP and halt operations until owner resolves the curse. In this case the prize risks to be locked forever in source chain, as you might have to re-send tx from source chain yet the functions that do this in source chain are only callable once per raffle.

### Root Cause

You should be able to re-send some messages from Avalanche to Ethereum in case of re-orgs during CCIP txs. This is only 1 way because finalization time in Avalanche is milliseonds and block creation time is around 2s, so the likelyhood of a re-org is very low or actually non-existant.

### Internal pre-conditions

Does not apply.


### External pre-conditions

- `Commitment-DON` executes tx in destination chain but latter a re-org occures and that tx gets errased.

- `RMN` and owner of RMN can only resolve the curse, which means lifting it and can't change the commitment on destination.

### Attack Path

#### 1️⃣ cancelRaffle()
1. Admins `cancelRaffle()` on Avalanvche and a CCIP message is sent to ETH. That `raffleID` in Avalanche is marked as `CANCELLED` (see [here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicketManager.sol#L281)) so future calls to `cancelRaffle()` will revert [here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicketManager.sol#L437). This tx reaches finality in Avalanche.
2. Now `Commitment-DON` see that it is safe to commit the txs from source chain to destination chain and it does it so. Yet `Commitment-DON` only commits once and does not care about re-orgs in destination chain.
3. A re-org happens in destination chain and removes the tx where `Commitment-DON` commited.
4. `RMN` (`Risk-Magement-Network-DON`), will see discrepancies in commitments in source and destination chain and curse CCIP halting operations until owner resolves the curse.
5. All is resolved and operations are back to normal, yet the commitment is not there anymore. So you have to re-send the `cancelRaffle()` tx from Avalanche.
6. You cant re-send a `cancelRaffle()` tx from Avalanche as now the raffle is marked in Avalanche as `CANCELED` from step `1`. Prize is locked forever in ethereum.

#### 2️⃣ propagateRaffleWinner()
1. Admins `propagateRaffleWinner()` on Avalanvche and a CCIP message is sent to ETH. That `raffleID` in Avalanche is marked as `PROPAGATED` [here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicketManager.sol#L337) so future calls to `propagateRaffleWinner()` will revert [here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesTicketManager.sol#L336). 
2. Same steps from previous example until step 5.
5. Now you try re-send the tx yet it reverts as the raffle is marked as `PROPAGATED` in Avalanche. Prize is locked forever in ethereum.

### Impact

Prizes locked forever, beause of unablity to cancel raffles or for a winner to claim them.


### PoC

This is based on how CCIP can work, in their docs [here](https://docs.chain.link/ccip) they have a purposely trained AI to answer questions about the code. These are the question you can ask them to verify that all assumptions that lead to this problem about CCIP.

I've asked chainlink devs about this, yet I have not received an answer and the contest ends tomorrow.

The AI is in their search bar at top right of the screen, you should be able to press `Ctrl + K` to open it if you can't find it.

Copy pate the questions on their AI, they will provide also links in the docs related to the answers. The AI should answer something similar to what it did to me:

```plaintext
QUESTION:

Commitment-DON submits the Merkle root commitment to the destination chain, yet there is a chance of a re-org happening on the destination chain. The RMN will detect this as the roots differ between the source and destination chains, causing CCIP to halt operations. Later, the owner of the RMN will decide to resolve the issue when all problems seem to have been mitigated. After that, everything will be working again, but the commitment sent by the Commitment-DON will still be erased from the destination chain along with the transaction it represented. Therefore, the senders of those transactions will need to re-send them from the source chain again.

ANSWER:
Based on the information provided in the knowledge sources, the Chainlink Cross-Chain Interoperability Protocol (CCIP) has a robust system in place to handle potential issues such as reorganizations on the destination chain.
The Risk Management Network (RMN) is a secondary validation service that monitors all Merkle roots of messages committed on each destination chain by the Committing Decentralized Oracle Network (DON). It independently reconstructs the Merkle tree by fetching all messages on the source chain and checks for a match between the Merkle root committed by the Committing DON and the root of the reconstructed Merkle tree. If both Merkle roots match, the RMN blesses the root to the Risk Management contract on the destination chain.
However, if the RMN detects an anomaly, such as a deep reorganization which violates the safety parameters set by the Risk Management configuration (finality violation), it will curse the CCIP system. After a quorum of votes has been met, the Risk Management contract dubs the CCIP system cursed. CCIP will automatically pause on that chain and wait until the contract owner assesses the situation before potentially lifting the curse.
The knowledge sources do not provide specific information on what happens to the commitments sent by the Committing DON that were erased from the destination chain along with the transaction they represented due to a reorganization. However, given the robustness of the system, it's reasonable to assume that there are mechanisms in place to handle such situations. For a more accurate answer, it would be best to refer to the official Chainlink documentation or reach out to the Chainlink team.

QUESTION 2:
Okay so lets break it down in simpler questions to asses the possibility. Does the commitment-DON only send the commitment once? This meaning if re-org happens in destination chain it will not send the commitment again. Second question: does RMN or its owner have the ability to change the merkle root commitment on destination chain?

ANSWER 2:
Based on the information provided in the knowledge sources:
The Committing Decentralized Oracle Network (DON) monitors events from a given OnRamp contract on the source blockchain, waits for finality, bundles transactions, creates a Merkle root, and writes the Merkle root to the CommitStore contract on the destination blockchain. The knowledge sources do not specify whether the Committing DON will resend the commitment if a reorganization happens on the destination chain. However, it's reasonable to assume that the Committing DON would need to monitor the finality of the destination blockchain and react accordingly if a reorganization occurs. For a definitive answer, it would be best to refer to the official Chainlink documentation or reach out to the Chainlink team. Source
The Risk Management Network (RMN) monitors the Merkle roots committed by the Committing DON into the Commit Store. Each node in the RMN compares the committed Merkle roots with the transactions received by the OnRamp contract. After verification, it calls the Risk Management contract to "bless" the committed Merkle root. In case of anomalies, each Risk Management node calls the Risk Management contract to "curse" the system. If the cursed quorum is reached, the Risk Management contract is paused to prevent any CCIP transaction from being executed. The knowledge sources do not specify whether the RMN or its owner has the ability to change the Merkle root commitment on the destination chain.

QUESTION 3:
Is it a viable scenario that given a discrepancy in merkle roots due to re-org in destination chain the RMN curses CCIP and the owner decides to cancel all commitments thus all transactions represented on them have to be re-sent from source chain? 

ANSWER 3:
Based on the information provided in the knowledge sources, the Risk Management Network (RMN) in Chainlink's Cross-Chain Interoperability Protocol (CCIP) has the ability to "curse" the CCIP system if it detects an anomaly, such as a discrepancy in Merkle roots due to a reorganization on the destination chain. This would cause the CCIP system to pause on that chain.
The owner of the Risk Management contract then has the responsibility to assess the situation. If the owner is satisfied that the issues have been resolved, they can revoke the cursing on behalf of Risk Management nodes. However, the knowledge sources do not specify whether the owner has the ability to cancel all commitments, nor do they provide information on what happens to the transactions represented by those commitments.
If a reorganization on the destination chain leads to the cancellation of all commitments, it's plausible that the transactions would need to be re-sent from the source chain. However, this is speculative and for a definitive answer, it would be best to refer to the official Chainlink documentation or reach out to the Chainlink team.
```

As you can see the risk is not assured yet its underlying logic makes sense and according to docs, it is plausible. 

Despite all this the unability to re-send txs when they could be actually done just in case a complex protocol like CCIP ends up not being able to deliver the message is essential in this system. As if any small problem arises from normal CCIP operations that ends up with no delivery on the other chain, the prize will be locked forever.

### Mitigation

Allow for cancellation of `CANCELED` raffles on avalanche yet not in mainnet. This should not supose any risk as the cancel function only changes 1 state variable in the contract which is the raffle status. Changing and already `CANCELED` status to `CANCELLED` will not have any risk.

```diff
    function _checkShouldCancel(uint256 raffleId) internal view {
        Raffle storage raffle = _raffles[raffleId];
        if (raffle.status == RaffleStatus.PRIZE_LOCKED) return;
+       if (raffle.status == RaffleStatus.CANCELED && callerIsAdmin) return;
        if (raffle.status != RaffleStatus.IDLE) revert InvalidRaffle();
        if (raffle.endsAt > block.timestamp) revert RaffleIsStillOpen();
        uint256 supply = IWinnablesTicket(TICKETS_CONTRACT).supplyOf(raffleId);
        if (supply > raffle.minTicketsThreshold) revert TargetTicketsReached();
    }
```

Notice that if the 1st cross-chain tx succeeded other cross-chain txs with the same `raffleID` will revert [here](https://github.com/sherlock-audit/2024-08-winnables-raffles/blob/main/public-contracts/contracts/WinnablesPrizeManager.sol#L282) as the raffle is marked as `CANCELED` in `Ethereum` from the 1st cross-chain tx. Note that you should not allow anyone to re-send a cross-chain tx as a malicious uer could just send this tx a lot of times and consume your `LINK` balance for useless txs so this re-sending tx should be a special admin role.

All same applies to the `progateRaffleWinner()` function, you should allow for re-sending of this tx in case of re-orgs or any CCIP issue. Notice that in this case the consumption of `LINK` has to be avoided in different ways as by the protocol rules admin should have no control over if the winner receives its price. My recommendation here is to have another variable that gives X tries of re-sending the tx. Re-orgs and multiple continue issues with CCIP are not that frequent and allowing for 5 re-sends should be enough, or maybe just charging the re-sender some money to avoid the consuming funds risk is another mitigation option.

```diff
    function propagateRaffleWinner(address prizeManager, uint64 chainSelector, uint256 raffleId) external {
        Raffle storage raffle = _raffles[raffleId];
+       if (raffle.status != RaffleStatus.FULFILLED || raffle.status != RaffleStatus.PROPAGATED) revert InvalidRaffleStatus();
+        if(numOfReSend > MAX_NUM_OF_RESENDS) revert MaxNumOfResendsReached();
        raffle.status = RaffleStatus.PROPAGATED;
        address winner = _getWinnerByRequestId(raffle.chainlinkRequestId);

        // Only fails if default gas sent is reduced maybe
        _sendCCIPMessage(
            prizeManager, chainSelector, abi.encodePacked(uint8(CCIPMessageType.WINNER_DRAWN), raffleId, winner)
        );
        IWinnablesTicket(TICKETS_CONTRACT).refreshMetadata(raffleId);
        unchecked {
            _lockedETH -= raffle.totalRaised;
        }
    }
```

This way even if a re-org occures, you can still re-send a cross-chain tx to unlock the prize or set the winner.

> ⚠️ **Note** 🚧 I'm not completely sure about the 1 direction mentioned in the **Root** section. It makes sense that re-org risk in `Avalanche` is minimal yet just in case similar measures can be applied in that direction of message sending. Competition time is limited and I could not research further, I encourage the dev team to do so or actually implementing these measures in the other direction to on the `lock` functions that send messages `Ethereum->Avalanche`.
