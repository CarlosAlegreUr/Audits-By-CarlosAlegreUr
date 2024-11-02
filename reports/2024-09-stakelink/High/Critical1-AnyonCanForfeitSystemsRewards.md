## Vulnerability Details

In Chainlink Staking v0.2 there is a **ramp-up period** when withdrawing rewards. This means that after the stake action has taken place there is a period of time when the rewards will slowly (linearly) become available. Currently, is 90 days.

The problem starts with: if you `claimRewards()` before the **ramp-up period** has passed, you will forfeit the rewards who have not been released yet, and these will go to other stakers on the pool.

Then on top of that we see on `CommunityVCS` that anyone can call `claimRewards()` anytime. Sure there is a `_minRewards` parameter but the attacker can just set it to 1 juel. See funtion [here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/linkStaking/CommunityVCS.sol#L125).

So, some staker, specially an individual who stakes on Chainlink pool yet not through **stake.link** can just call this `claimRewards()` every time a juel is accrued and make the protocol forfeit the rest, which in this case, as the attacker is an independent staker, part of the forfeit will go to him.

You can read [here](https://blog.chain.link/chainlink-staking-v0-2-overview/#claimable_rewards_and_ramp-up_period) more about the **ramp-up period** and forfeiting rewards.

## Impact

Anyone can make stakers from **stake.link** forfeit their rewards and indeed there is even a incentive to do so if you are an independent staker.

## Recommendations

You can opt for different approaches to fix this:

1️⃣ Check for the **ramp-up** period before allowing `claimRewards()`. You can do so with the `getMultiplier()` in the Chainlnk Rewards contract. Do not allow anything under 100% of completed ramp-up to be claimed.

2️⃣ Allow for each value to have a ramp-up tolerance value. This means lets say that vault doesn't mind if 20% of rewards are forfeited, then at the time of checking the ram-up, if below 80%, revert.

3️⃣ Add access control to `claimRewards()`. Only allow the vault or trusted addresses to call it.
