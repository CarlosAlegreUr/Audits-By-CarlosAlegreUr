## Summary

Lack of slippage during claiming can lead to stakers unexpectedly losing extra rewards.

## Vulnerability Detail

The amount a staker receives when claiming is deducted a fixed fee. See [here](https://github.com/sherlock-audit/2024-11-tally/blob/main/staker/src/GovernanceStaker.sol#L720).

If the fees eat up all the staker's rewards the function just [`returns 0;` in the next line of code](https://github.com/sherlock-audit/2024-11-tally/blob/main/staker/src/GovernanceStaker.sol#L721). Yet any unclaimed rewards tracking is updated and no fee transfer is made. This is fine as the staker has no rewards to claim.

The problem comes when a user accidentally claims his rewards right after a fee update that will eat more than he desired yet not drain them to 0. The staker will lose more than he expected.

We can discard the admin griefing the staker with fee front-run or MEV like that as admin is trusted.

Yet we cannot discard the possibility of just accidentally a bad timing for the staker with some fee update where his tx ends up right after a fee increase update. Furthermore, in Arbitrum the mempool is not public so it is pretty difficult for a staker to avoid bad timing.

On top of all this, rewards are also affected by bump fees, [see here](https://github.com/sherlock-audit/2024-11-tally/blob/main/staker/src/GovernanceStaker.sol#L512). Multiple fees can be applied by system actions without giving the staker the chance to react on the slippage they generate.

This is why a fee slippage is needed.

## Impact

Stakers can unnecessarily receive fewer rewards that expected due to fee slippage. Notice that the staker if he would've been protected from this he might have just not claimed and waited until the admin lowers down the fees in the future. Avoidable, unexpected loss of funds.

## Tool used

Manual Review

## Recommendation

Add an `expetedRewardsAfterFees` parameter set by the sender of the tx while claiming fees.
