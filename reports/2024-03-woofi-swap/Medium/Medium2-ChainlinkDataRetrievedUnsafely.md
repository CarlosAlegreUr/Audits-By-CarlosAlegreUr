### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

## Summary

When checking Chainlink Data Feeds (as per their own Chainlink docs, direct links provided at the end) you have to check 2 things to make sure the data you are getting is reliable. The protocol checks none.

Although a fail in the Chainlink network is rare, it can happen and it is something your code have to be ready for.

## Vulnerability Detail

1️⃣ You have to check how long its been since the last update. If its been too long, the data provided might be stale and should not be used as users would be getting unfair high or low prices.

The code uses `latestRoundData()` to get the price of the `base` and `quote` assets:

```solidity
(, int256 rawBaseRefPrice,, uint256 baseUpdatedAt,) = AggregatorV3Interface(baseOracle).latestRoundData();
(, int256 rawQuoteRefPrice,, uint256 quoteUpdatedAt,) = AggregatorV3Interface(quoteOracle).latestRoundData();
```

But, each feed on its own and in different chains can be updated at different rates (Chainlink calls them `Heartbeat`). For `ETH/USD ArbitrumOne`, the data feed is given new data every 3600s => 1h. As you can see [here](https://docs.chain.link/data-feeds/price-feeds/addresses?network=arbitrum&page=1&search=ETH%2FUSD) in the Chainlink docs, down in the **Pairs** table clicking on the `[] Show more details` checkbox.

In the ETH/USD case, if the price feed has not been updated in the last hour an incorrect price can be used whether for good or bad as this can harm users given outdated more expensive prices or benefit them giving outdated cheaper prices, though selling cheaper assets could damage the protocol.


2️⃣ In the case of some L2, like **Arbitrum**, when uing Chainlink Price Feeds you must check wheter the sequencer is up or down because a sequencer down means the data could be stale but the previous check has not been updated yet. This also applies for other 2 chains the protocol is also deployed on: **Optimisim** and **Base**.

## Impact

Stale data can damage the users and the protocol giving unfair prices for any of them.

## Code Snippet

Here you can see the `AggregatorV3Interface(baseOracle).latestRoundData();` calls and how there is no checks on chainlink data staleness:

https://github.com/sherlock-audit/2024-03-woofi-swap-CarlosAlegreUr/blob/main/WooPoolV2/contracts/wooracle/WooracleV2_2.sol#L360

## Tool used

- Manual audit.

## Recommendation

1️⃣ To avoid scenario 1 you can use the `uint256 updatedAt` return data from `AggregatorV3.latestRoundData()`. You should check `block.timestamp - updatedAt >= heartbeatOfDataFeed` to see if its been too long since the last price update before proceeding to use the data. And then, according to that, handle the situation as you see fit, probably with a revert if accurate data can't be provided.

Chainlink Docs example explaining this: [click here](https://docs.chain.link/data-feeds/l2-sequencer-feeds)

2️⃣ There is a special contract by Chainlink that checks for downtime of the sequencer in L2s. Check the Chainlink docs for more info: [see exact docs here](https://docs.chain.link/data-feeds/l2-sequencer-feeds).
