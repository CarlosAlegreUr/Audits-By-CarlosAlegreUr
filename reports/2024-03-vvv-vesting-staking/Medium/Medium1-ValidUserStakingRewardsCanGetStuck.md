### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

## Summary

Users can get their **VVV** staking rewards stuck due to a DOS which happens due to the claim tx hitting the block gas limit.

## Vulnerability Detail

Every time a user stakes eth eventually `_stakeEth()` is called then the `_userStakeIds` array get pushed a new element: `_userStakeIds[msg.sender].push(stakeId);`

```solidity
    ///@notice Private function to stake ETH, used by both stakeEth and restakeEth
    function _stakeEth(StakingDuration _stakeDuration, uint256 _stakedEthAmount) private {
        // more code...
        _userStakeIds[msg.sender].push(stakeId); // 🟢 <-- See here
        //more code..
```

When calling `claimVvv()` the `for` loop in `calculateAccruedVvvAmount()` will loop through all `stakeIds.length` which comes from: `stakeIds = _userStakeIds[msg.sender];`. If user has done multiple stakes then the foor loop will consume all gas and revert all the `claimVvv()` that the user tries to execute.

## Impact

Users will not be able to claim their rewards if they stake several times.

## Code Snippet

- Push that increments the array: [click to see here.](https://github.com/sherlock-audit/2024-03-vvv-vesting-staking/blob/main/vvv-platform-smart-contracts/contracts/staking/VVVETHStaking.sol#L301)

- Loop that consumes all gas when claiming `VVV`: [click to see here.](https://github.com/sherlock-audit/2024-03-vvv-vesting-staking/blob/main/vvv-platform-smart-contracts/contracts/staking/VVVETHStaking.sol#L206)

- `claimVvv()` calls `calculateClaimableVvvAmount()`, which calls  `calculateAccruedVvvAmount()`.
See here first call: [click to see here.](https://github.com/sherlock-audit/2024-03-vvv-vesting-staking/blob/main/vvv-platform-smart-contracts/contracts/staking/VVVETHStaking.sol#L186)
See here second call: [click to see here.](https://github.com/sherlock-audit/2024-03-vvv-vesting-staking/blob/main/vvv-platform-smart-contracts/contracts/staking/VVVETHStaking.sol#L250C16-L250C41)


## Tool used

Manual Review

## Recommendation

One way of solving it could be switching to a `claimReward(stakeId)` way of claiming rewards. This would claim rewards of a specific stake action by the user. Then if desired to do more than 1 in 1tx you could create a `batchClaimRewards(uint256[] stakeIds)` so its more comfortable and gas saving.
