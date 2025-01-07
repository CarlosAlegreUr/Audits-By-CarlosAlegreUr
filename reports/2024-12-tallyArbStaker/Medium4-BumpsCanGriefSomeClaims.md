## Summary

Updates from the calculator can be griefed by a bumper if the claimer is going to claim in the same block.

## Vulnerability Detail

The whole purpose of `bumpEarningPower()` is to have an incentive to properly and quickly update the systems' `totalEarningPower`.

This is necessary because some deposits might be untouched for a long time and if they are not touched their earning power will remain the same even though it can be outdated and have already changed in the calculator.

So all positions which had their earning power changed in the calculator and require an update should either be touched (any action triggers the update, [even changing the claimer](https://github.com/sherlock-audit/2024-11-tally/blob/main/staker/src/GovernanceStaker.sol#L661)) or bumped to properly keep track of the `totalEarningPower`.

It makes sense and it is fair to reward the bumper with a small part of the rewards accrued as it is necessary for a healthy functioning of the system.

But, there is a scenario where it is unfair and griefs the depositor and/or claimer. This is when in the same block:

- Calculator updates the earning power of the delegatee.
- Depositor is going to claim rewards. His tx will be placed after the notification, yet no time really passed from an inactive outdated deposit that must be updated.
- Bumper sees this and decides to front-run and bump him.

This creates an unfair MEV from which the claimer should be protected, as per what tx ordering timing regards, he was going to update his position on time and keep the system healthy, he must not be punished for this and he should be protected. The MEV is the following:

As the calculator sends a new earning score which will require an update of the `totalEarningPower` a depositor is going to `claimReward()` thus triggering that update.

Yet a bumper sees this and front-runs the claimer to grief him from his rewards, sandwiching his transaction between the update and the claim in the same block.

## Impact

A claimer gets griefed by the bumper the bump tip from his rewards.

## Tool used

Manual Review

## Recommendation

Only allow any bumping after 1 timestamp unit passed since the time when the calculator updated the delegatee's score.
