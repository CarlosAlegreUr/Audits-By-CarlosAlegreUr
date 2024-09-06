## Relevant Context

Question to the devs [here](https://cantina.xyz/code/80a33cf0-ad69-4163-a269-d27756aacb5e/comments#comment-a3d670ee-cb03-45eb-ab89-d4f28db2b5b3).

![Screenshot from 2024-08-27 09-59-20.png](https://imagedelivery.net/wtv4_V7VzVsxpAFaxzmpbw/c8926d9b-26f9-463f-fbd3-4570fee26a00/public) 

## Description

There is no `depositAndRevert()` for Zeta token in `GatewayZEVM`. The only one implemented is for ZRC20s, yet Zeta token is not a ZRC20 in Zeta chain.

## Impact Explanation

If a user sends Zeta token from any spoke chain and it reverts, the user won't be able to use the `onRevert()` hook if desired as there is just no implementation for that with Zeta token in the `GatewayZEVM` contract. As said from devs, this implementation is needed, intended and missing.

## Recommendation

Create the other `depositAndRevert()` for Zeta token in GatewayZEVM.
