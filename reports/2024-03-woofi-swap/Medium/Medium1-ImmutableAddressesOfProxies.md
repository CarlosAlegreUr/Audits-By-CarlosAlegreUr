### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

## Summary

Third party proxies are used as immutable addresses in the protocol. If the proxy is compromised, the system would have no way of changing the address but a costly pause, get all users' funds and carefully redeploy which is prone to errors.

## Vulnerability Detail

Along the codebase multiple addresses which are PROXYs in some blockchains are used as `immutable` variables.

For example in `WooPPV2.sol` and in `WooRouterV2.sol` there are `immutable addresses` for tokens that are a Proxy.

Should anything bad happen to that third party PROXY and the system would have no way of changing the address. And these are key addresses that manage funds and are used in essential parts of the protocol like the `swap()` function on `WooPPV2`.

For example in **Arbitrum** the **WETH** token is behind a proxy. If the proxy is compromised, the system would have no way of changing the address but a costly pause, get all users' funds and carefully redeploy which is prone to errors.

Same goes for some stablecoins the protocol uses as quote tokens like **USDC** on **Arbitrum**.

## Impact

The deployed code would deem unusable and the protocol would need to be paused and carefully redeployed.

## Code Snippet

- [quoteToken immutable code on WooPPV2](https://github.com/sherlock-audit/2024-03-woofi-swap/blob/main/WooPoolV2/contracts/WooPPV2.sol#L86)
- [WETH immutable code on RouterV2](https://github.com/sherlock-audit/2024-03-woofi-swap/blob/main/WooPoolV2/contracts/WooRouterV2.sol#L61)

- [Click here](https://arbiscan.io/address/0x28D2B949024FE50627f1EbC5f0Ca3Ca721148E40#readContract) to see the `IntegrationHelper` contract on **Arbitrum** the protocol has already deployed which tells you which tokens the procotol uses. You can see among them **USDC** which is a PROXY.

## Tool used

Manual Review

## Recommendation

As your protocol already gives the owner a lot of power, won't be much worry if now these addresses are part of the contract state and have a setter function with an `onlyOwner` modifier.

Check on all blockchains that the protocol is meant to be deployed that there are no immutable addresses behind a third-party's proxy.