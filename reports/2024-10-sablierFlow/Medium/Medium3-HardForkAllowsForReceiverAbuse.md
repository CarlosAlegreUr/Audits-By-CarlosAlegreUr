## Vulnerability Details

Imagine a new version of a chain is created by a hard-fork, or just a hard-fork happens, and **the `sender` (business) wants to operate in both chains**.

Imagine a `Stream` is being used for a payroll to an employee thus you only want to pay him once.

Then a hard-fork occurs and this opens the possibility for the `recevier` to withdraw the funds twice from the `Stream`, one in the first chain and another one in the forked one. Effectively receiving its payroll twice.

## Impact

In case of hard-fork `receiver` can effectively double-withdraw the `Stream` without previous `sender` consent. The protocol assumes there has been consent previous to the stream creation, yet here the stream is created by none of the parties but as a result of external forces like a hard-fork, thus, we can say that this new **Stream** can't be considered a consensual one and must not be treated as such.

## Recommendations

Add on the [`Stream` struct](https://github.com/Cyfrin/2024-10-sablier/blob/main/src/types/DataTypes.sol#L61) an extra parameter, the agreed `chainId` where the `Stream` is valid between parties. In every action, if the `chainId` changed and the `Stream` is not valid in the new chain, then the action should revert.
