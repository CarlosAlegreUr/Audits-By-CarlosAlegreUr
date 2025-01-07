## Summary

Bumping yourself allows you to skip the claiming fee.

## Vulnerability Detail

When `bumpEarningPower()` a fee is taken from the unclaimed rewards and given to the bumper as an incentive to keep the system always updated. See [here](https://github.com/sherlock-audit/2024-11-tally/blob/main/staker/src/GovernanceStaker.sol#L511).

The bumper can perfectly be the very same depositor or claimer of the deposit. Thus, the fee taken from the unclaimed rewards is still transferred to them in that case. Effectively claiming their reward without having to call `claimReward()` which would have charged them a fee from their claiming reducing their overall earnings.

If they do this process enough times they can claim all their rewards without ever having to pay the fee for it.

## Impact

As of the present calculator you can only skip your fee every time the oracle updates the score of your delegatee.

It is reasonable to expect the score changing, in any kind of calculator, every time significant changes in voting power or voting token balance happen to a delegatee. Thus the depositor would only have to do the following to repeatedly claim their rewards without paying the fee:

- Transfer voting power or tokens to delegatee ( delegatee can be the very same depositor, as the only condition against the delegatee address is, [this one](https://github.com/sherlock-audit/2024-11-tally/blob/main/staker/src/GovernanceStaker.sol#L563)), triggering oracle update.
- Bump earning power, skip a fee.
- Transfer again.
- Bump earning power, skip a fee.
- And so on.

## Proof of concept

Bumping is always executed [if](https://github.com/sherlock-audit/2024-11-tally/blob/main/staker/src/GovernanceStaker.sol#L476):

```solidity
  function bumpEarningPower(/*args*/) external virtual {
@>  // 👁️🟢1️⃣ Bumper puts a lower tip than the max allowed. Then this does not revert.
    if (_requestedTip > maxBumpTip) revert GovernanceStaker__InvalidTip();

    // more code...

@>  // 👁️🟢2️⃣ New delegate got a different earning power.
    (uint256 _newEarningPower, bool _isQualifiedForBump) = earningPowerCalculator.getNewEarningPower(
      deposit.balance, deposit.owner, deposit.delegatee, deposit.earningPower
    );

@>  // 👁️🟢3️⃣ Thus _isQualifiedForBump==true and new earning is different to the previous one. This does not revert.
@>  // Even in the case of the current calculator where _isQualifiedForBump==false, the bumper just has to wait a delay and then execute.
    if (!_isQualifiedForBump || _newEarningPower == deposit.earningPower) {
      revert GovernanceStaker__Unqualified(_newEarningPower);
    }

@>  // 👁️🟢4️⃣ New earning is higher and you just make the tip smaller than the already unclaimed rewards. This does not revert.
    if (_newEarningPower > deposit.earningPower && _unclaimedRewards < _requestedTip) {
      revert GovernanceStaker__InsufficientUnclaimedRewards();
    }

@>  // 👁️🟢4️⃣ If the earning is lower due to the naughty transfer you can still tweak the tip so this doesnt revert. All executes.
    if (_newEarningPower < deposit.earningPower && (_unclaimedRewards - _requestedTip) < maxBumpTip)
    {
      revert GovernanceStaker__InsufficientUnclaimedRewards();
    }

    // more code...
  }
```

## Tool used

Manual Review

## Recommendation

Do not allow claimer or depositor to bump themselves, or discount from the tip part of the claim fee if they are the bumpers.
