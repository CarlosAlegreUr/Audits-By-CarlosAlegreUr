## Description

When sending jut a call, `call()`, from `Zeta -> SpokeChain` there is an empty check for the message and reverts if message is empty. It does not so the other way around, `call()` from `SpokeChain -> Zeta`.

## Impact Explanation

Allowing empty messages makes no logical harm to the state of the blockchains, yet it is useless and wasted network resources. Even if the observer sees them and discards them that is still a waste of network resources. Also it can potentially be leveraged by attackers who seek to overload the network cheaply.

## Recommendation

As done in `call()` at `GatewayZEVM`, rever at `call()` in `GaetewayEVM` if: `if (message.length == 0) revert EmptyMessage();`

