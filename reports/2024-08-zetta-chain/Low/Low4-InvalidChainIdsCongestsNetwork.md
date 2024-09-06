## Description

The protocol allows to send Zeta tokens with `Zeta -> Spoke` transactions with any kind of `chainId`. Even if Zeta plans to be universal, until that time comes not all chain ids will be supported.

## Impact Explanation

If someone just sends a tx with an un-supported `chainId` I guess the observer will just refund the user. Yet this is unnecessary overload on the network and can be leveraged by attackers to overload the network cheaply.

## Recommendation

Create a `mapping(chaiId => isSupported)` in `GatewayZEVM` and revert if not supported. This will prevent event to be emitted and unnecesary overload on the network.

