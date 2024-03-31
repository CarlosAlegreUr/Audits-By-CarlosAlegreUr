### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

## Summary

The `ethToVvvExchangeRate()` function returns a constant value when it will likely be necessary to change it in the future as the very same team has mention in the docs provided: [see docs](https://hackmd.io/@vvv-knowledge/Syme5HlRT#Accrued-VVV-calculation).

## Vulnerability Detail

In the docs mentioned in the Sherlok README there is this link: 

```txt
Add links to relevant protocol resources

https://hackmd.io/@vvv-knowledge/Syme5HlRT
```

If you go there and go to the last section [Accrued $VVV calculation](https://hackmd.io/@vvv-knowledge/Syme5HlRT#Accrued-VVV-calculation) it clearly states that the `ethToVvvExchangeRate` is currently set to 1 and is therefore redundant but this will almost certainly change in the future.

However the way is set is via a constant value returned by a pure function. Whenver it needs to be changed it wont be posible.

## Impact

In the likely scenario of needing to change the essential parameter that regulates staking rewards the team wont be able to do so. 

## Code Snippet

See the function in the repo [clicking here](https://github.com/sherlock-audit/2024-03-vvv-vesting-staking/blob/main/vvv-platform-smart-contracts/contracts/staking/VVVETHStaking.sol#L254).

See part of the code where its called when calculating rewards [clicking here](https://github.com/sherlock-audit/2024-03-vvv-vesting-staking/blob/main/vvv-platform-smart-contracts/contracts/staking/VVVETHStaking.sol#L237).

```solidity
    ///@notice Returns the exchange rate of ETH to $VVV for staking reward accrual
    function ethToVvvExchangeRate() public pure returns (uint256) {
        return 1;
    }
```

## Tool used

Manual Review

## Recommendation

Create a state variable for the exchange rate only modifiable by the admin. As this is a crucial parameter only the admin should be able to change it, and also, admin already has strong privileges in the overall system and is considered `TRUSTED` so giving him the ability to change this parameter should not be a problem.
