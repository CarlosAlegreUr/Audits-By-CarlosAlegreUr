## Vulnerability Details

`OperatorStakingPool.sol` is meant to hold LST from operators. It receives them from an LST ERC677 `transferAndCall()` action. Yet the contract has no way of transferring those LST out of it.

There are no transfers neither approvals to other protocol contracts to use a `transferFrom()` and so on. Funds get stuck.
See the contract and the specific `_withdraw()` logic [here](https://github.com/Cyfrin/2024-09-stakelink/blob/main/contracts/linkStaking/OperatorStakingPool.sol#L202).

## Impact

Operators who send their LST to the `OperatorStakingPool` are stuck forever.

## Recommendations

In the very same contract, on the `withdraw()` function, actually transfer them out.