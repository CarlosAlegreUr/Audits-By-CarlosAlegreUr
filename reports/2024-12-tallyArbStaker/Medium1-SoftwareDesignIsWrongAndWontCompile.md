## Summary

The design of `GovernanceStaker` contract is wrong. Users using the code of 2 extensions won't even compile.

These are `GovernanceStakerOnBehalf` and `GovernanceStakerPermitAndStake`.

## Vulnerability Details

There is a significant error in the code design. The docs of `GovernanceStaker` contract say:

```solidity
/// The contract allows stakers to delegate the voting power of the tokens they stake to any
/// governance delegatee on a per deposit basis. The contract also allows stakers to designate the
/// claimer address that earns rewards for the associated deposit.
```

Yet that functionality is added on the `GovernanceStakerDelegateSurrogateVotes` when overriding `_fetchOrDeploySurrogate()` and `surrogates()`.

The other contracts of the code: `GovernanceStakerOnBehalf` and `GovernanceStakerPermitAndStake` add signature-based access to the staking functionalities yet by their own they will not work as the 2 before-mentioned functions will remain empty. This is because [they inherit from `GovernanceStaker`](https://github.com/sherlock-audit/2024-11-tally/blob/main/staker/src/extensions/GovernanceStakerPermitAndStake.sol#L14) and not from `GovernanceStakerDelegateSurrogateVotes`.

## Impact

The code design is not what it was intended. Tests only pass because the harness inherits all the contracts including a `GovernanceStakerDelegateSurrogateVotes` contract which actually overrides the functions.

These functions are pivotal to the code as they are the ones who guarantee that staked tokens can still be used for voting.

> ℹ️ **Note** 📘 I don't know how this kind of issue is judged on Sherlock. Implementation is wrong to the point of software design and it won't even compile.

## Proof Of Concept

Paste this contract on `src/extensions/`, your linter should warn you that virtual overriding functions are missing and won't compile:

```solidity
// SPDX-License-Identifier: AGPL-3.0-only
pragma solidity ^0.8.23;

import {GovernanceStakerPermitAndStake} from "./GovernanceStakerPermitAndStake.sol";
import {IERC20Permit} from "openzeppelin/token/ERC20/extensions/IERC20Permit.sol";
import {GovernanceStaker} from "src/GovernanceStaker.sol";
import {IERC20} from "openzeppelin/token/ERC20/IERC20.sol";
import {SafeERC20} from "openzeppelin/token/ERC20/utils/SafeERC20.sol";
import {IERC20Staking} from "src/interfaces/IERC20Staking.sol";
import {IEarningPowerCalculator} from "src/interfaces/IEarningPowerCalculator.sol";


contract ContractImplementing is GovernanceStakerPermitAndStake {
    constructor(
        IERC20 _rewardsToken,
        IERC20Staking _stakeToken,
        IEarningPowerCalculator _earningPowerCalculator,
        uint256 _maxBumpTip,
        address _admin,
        string memory _name
    )
        GovernanceStakerPermitAndStake(_stakeToken)
        GovernanceStaker(
            _rewardsToken,
            _stakeToken,
            _earningPowerCalculator,
            _maxBumpTip,
            _admin
        )
    {}
}
```

## Recommendation

Make the contracts inherit from `GovernanceStakerDelegateSurrogateVotes` instead of `GovernanceStaker`.

Or make the 2 named virtual functions have the default implementation in `GovernanceStaker` and get rid of the `GovernanceStakerDelegateSurrogateVotes` contract.
