## Description

At `GatewayZEVM` in `withdraw()` and `withdrawAndCall()` you must specify the `chainId` of the spoke chain you wanna withdraw to. Yet if the chain is the Zeta chain itself nothing happens.

My take is that the `Observer` will complete the message by calling in the very same contract the `depositAndCall()` function to re-send the Zeta to Zeta chain to the `receiver`.

## Impact Explanation

This is clear waste of network resources and it also could be leveraged by attackers who seek to overload the network cheaply.

## Recommendation

At the withdraw function `if(chainId == block.chainid){revert;}`.
