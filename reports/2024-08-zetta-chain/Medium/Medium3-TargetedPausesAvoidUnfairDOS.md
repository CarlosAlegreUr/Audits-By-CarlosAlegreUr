## Description

The ability to pause all the system parts if necessary is a good feature. Yet it must be complemented with also some granular pausing because otherwise a fail in one part of the system can lead to a denial of service in the whole system. It is true that when a solution is found and fixed, everything can be resumed, but parts of the system that could have been operating perfectly will be pause indeterminately.

## Impact Explanation

An issue with a spoke chain that requires quickly pausing `GatewayZEVM` and `GatewayEVM` so as to people do not use the corrupted cross-chain pipe `Zeta -> SpokeChain` happens. This will cause undetermined DOS on all spoke chains, even if they have no issues.

Same with some ERC20 tokens needing pause. If any token needs pause the whole operations of `ERC20Custody` will be. Affecting other functional ERC20s for an undetermined time causing a DOS on all tokens.

Maybe the fix is quick and all can be resumed soon, and maybe the fix is really hard and it will take a long time. In that second case all the system value will be locked by the fault of 1 part of the system  unnecessarily penalizing the rest of the system with stuck funds.

## Recommendation

Complement the global pause with a more granular pause that allows pausing only the parts of the system that are affected by the issue. For example be able to pause function calls but only when they are directed to an affected spoke chain in `GatewayZEVM` or token in case of `ERC20Custody`.
