## Vulnerability Details

One of the main reasons Chainlink created its staking mechanism is to be able to further protect their networks with cryptoeconomic incentives. This is done like so by slashing operators of their stake if they misbehave.

The problem is that in `stake.link` operators put nothing of their own money as they lease their staking room to users. Thus the operator does not really lose much when slashing occurs as the slashed staked amount actually belongs to normal users.

## Impact

This weakens and undermines one of the core purposes of LINK staking and gives operators a weaker incentive to behave correctly. This is because the only thing they lose (or better said, stop earning) at slashing is the accrued rewards they would have gotten if the slashed capital would had stayed in the pool. Yet the loss of the capital itself is actually no harm to them.

## Recommendations

Create, as Chainlink has, an unbonding period for operators to claim their rewards. And, if they get slashed during that unbonding period, take them away and give them to the community.

This way the actual capital operators use the system for is at risk if they misbehave. 
